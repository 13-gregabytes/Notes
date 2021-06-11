# Remove a (large) file from a git commit
> https://medium.com/analytics-vidhya/tutorial-removing-large-files-from-git-78dbf4cf83a

## The Error Message
So, you just tried to run git push, and after taking longer than usual, you get an error trace like this one:
```
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com
remote: error: Trace: 08740bd2fb02f980041be67b73e715a9
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File csv_building_damage_assessment.csv is 218.83 MB; this exceeds GitHub's file size limit of 100.00 MB
To https://github.com/hoffm386/git-large-file-example.git
! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/hoffm386/git-large-file-example.git'
```
## What Happened?
When you tried to run git push, it failed. None of your changes have been pushed to GitHub, although nothing has changed locally. The reason the push failed is best highlighted by this line of the error message:

remote: error: File csv_building_damage_assessment.csv is 218.83 MB; this exceeds GitHub's file size limit of 100.00 MB

In my case the file was called csv_building_damage_assessment.csv, but any file larger than 100MB can cause this error (.zip, .pdf, .xlsx, .pkl, etc.). To quote from the GitHub documentation:

“GitHub limits the size of files allowed in repositories, and will block a push to a repository if the files are larger than the maximum file limit…GitHub blocks pushes that exceed 100 MB.”

GitHub provides a lot of services for free, but they generally charge money for storing and versioning large files through their Large File Storage product, and do not allow files larger than 100MB to be pushed to their standard repositories.

## What Not to Do
Often a beginner’s first intuition is just to make a new commit that deletes the large file, something like:
```
git rm csv_building_damage_assessment.csv
```
```
git commit -m "removing large file"
```
Unfortunately, this won’t work, since when you push something to GitHub, GitHub is actually promising to keep track of each any every commit, and allow you to roll back to any place in your history. So if you push a sequence of commits that adds then deletes a large file, that’s still asking GitHub to store the large file, and GitHub will still block the push. You need a solution that “rewrites history” to make it seem to GitHub that you never added the large file in the first place!

## What to Do
We need to amend (i.e. edit) the commit where the large file was added in order to “rewrite history”.

The number of commands required to do this depends on which commit added the large file. The two scenarios are:

The large file was just added in the most recent commit

The large file was committed prior to the most recent commit
>CAUTION: when you start rewriting history, accidentally running the wrong command means potentially deleting the large file. If the contents of the file are important to you, make a copy of the file outside of the repository (e.g. on your desktop) so that you can recover it later.

### Scenario 1: The Large File Was Just Added in the Most Recent Commit
Let’s start by addressing scenario 1, which is easier to handle.

In this scenario, you can amend the most recent commit to remove the large file. This is the same as any other action that amends the most recent commit: first you make the change, then you run git commit --amend. In the case of the example above, you would run the following in the terminal (at the root of your repository):
```
git rm --cached csv_building_damage_assessment.csv
```
```
git commit --amend -C HEAD
```
(Replacing csv_building_damage_assessment.csv with the name of your large file)

That’s it! The large file has been removed from the commit history, and you should now be able to push to GitHub.

### Scenario 2: The Large File Was Committed Prior to The Most Recent Commit
This situation is more complicated than the first one, but still fixable! There are multiple possible approaches, but I recommend an interactive rebase as the simplest approach that still maintains fine-grained control.

The explanation for this scenario is long enough that I’m going to add a whole new heading for it:

Interactive Rebase for Removing Large Files

Conceptually what we’re doing here is looking back through the Git history, finding the commit where the large file was added, and editing that commit while leaving the others alone.

Locating the Last “Good” Commit

Run this command in the terminal to print out the commit history of the repository:
```
git log
```
Your output might look slightly different based on your settings, but in general you should see something like this:
```
099e6e4 update gitignore to ignore large data file
de69e51 preliminary exploratory data analysis
d1bfae6 download data CSV
8464da4 update README
48f7303 Initial commit
```
When I look at this output, I can see that “download data CSV” was when I downloaded this large file. This is just one example of why it’s important to write meaningful commit messages! If you’re not sure which commit was the last “good” one, you’ll need to try this process repeatedly with different commits, until you find the right one.

So, now that I’ve identified the message of the last “good” commit as “update README”, I need to identify the commit hash. This is the unique identifier of the commit, in this case located to the left of the commit message. It’s highlighted in bold here:
```
099e6e4 update gitignore to ignore large data file
de69e51 preliminary exploratory data analysis
d1bfae6 download data CSV
8464da4 update README
48f7303 Initial commit
```
Initiate a Rebase Between the Last “Good” Commit and the Current Commit

Now that I’ve identified the commit hash, I run this command using that hash:
```
git rebase -i 8464da4
```
This will open up a file in your Git editor (in my case, Vim), that looks something like this:
```
pick d1bfae6 download data CSV
pick de69e51 preliminary exploratory data analysis
pick 099e6e4 update gitignore to ignore large data file
# Rebase 8464da4..099e6e4 onto 8464da4 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
There’s a lot of information in there, since an interactive rebase can be used for a lot of things, not just removing large files from history. The only lines in this file that actually matter are the first three, everything else is just providing instructions:
```
pick d1bfae6 download data CSV
pick de69e51 preliminary exploratory data analysis
pick 099e6e4 update gitignore to ignore large data file
```
Notice how the last “good” commit is not here, it’s just the commits that happened after that one. I happen to know that the commit causing the problem was the “download data CSV” one, so I’m going to edit the file to say that I want to edit that commit, and just pick (i.e. keep without changes) the other two:
```
edit d1bfae6 download data CSV
pick de69e51 preliminary exploratory data analysis
pick 099e6e4 update gitignore to ignore large data file
```
Then I save and close the file (`:wq` in Vim)

### Amending the Commit
Now that I’ve closed the file, I see this message in the terminal:
```
Stopped at d1bfae6...  download data CSV
You can amend the commit now, with
  git commit --amend
Once you are satisfied with your changes, run
  git rebase --continue
```
So now I’ll run essentially the same command as if I had only added the large file in the most recent commit:
```
git rm --cached csv_building_damage_assessment.csv
```
```
git commit --amend -C HEAD
```
If you made additional changes to the repository other than just adding the file, that’s all you need to do.

But if adding that CSV was literally the only thing you did in that commit, you might get this message:
```
interactive rebase in progress; onto 8464da4
Last command done (1 command done):
edit d1bfae6 download data CSV
Next commands to do (2 remaining commands):
pick de69e51 preliminary exploratory data analysis
pick 099e6e4 update gitignore to ignore large data file
You are currently splitting a commit while rebasing branch 'master' on '8464da4'.
Untracked files:
csv_building_damage_assessment.csv
No changes
You asked to amend the most recent commit, but doing so would make
it empty. You can repeat your command with --allow-empty, or you can
remove the commit entirely with "git reset HEAD^".
```
If that’s the case — you didn’t to anything except add a file or files too large for GitHub in this commit—you have a couple of options, including starting over (or using git rebase --edit-todo) and replacing pick with drop instead of edit. But because I like being able to see the original commit history, I recommend that you use the --allow-empty flag. In this case, that would mean:
```
git commit --amend --allow-empty -C HEAD
```
Now you should be done amending the commit (whether or not you had to re-run the command with --allow-empty). If you run git status, it will look something like this:
```
interactive rebase in progress; onto 8464da4
Last command done (1 command done):
edit d1bfae6 download data CSV
Next commands to do (2 remaining commands):
pick de69e51 preliminary exploratory data analysis
pick 099e6e4 update gitignore to ignore large data file
(use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'master' on '8464da4'.
(use "git commit --amend" to amend the current commit)
(use "git rebase --continue" once you are satisfied with your changes)
Untracked files:
(use "git add <file>..." to include in what will be committed)
csv_building_damage_assessment.csv
nothing added to commit but untracked files present (use "git add" to track)
Finishing the Rebase
Once the commit adding the large file is fixed, the last thing you need to do is finish the rebase with:
git rebase --continue
```
You should see an output like:
```
Successfully rebased and updated refs/heads/master.
```
Now you should be able to git push without any error messages about large files ✨

## Recap
This error message happens when you try to push a file larger than 100MB to GitHub. To fix this issue, you can’t just remove the file from future commits, you need to “rewrite history” and edit whichever commit introduced the large file.

If the large file was added in the most recent commit, you can just run:
```
git rm --cached <filename> to remove the large file, then
```
```
git commit --amend -C HEAD to edit the commit
```
If the large file was added in an earlier commit, I recommend running an interactive rebase. That means you need to:

Run git log to find the commit hash of the last commit before you added the large file

Then run git rebase -i <commit hash>. This will open up an editor where you want to replace pick with edit on the commit where the large file was added.

Once you save and close the editor, you’ll be in essentially the same position as if you had added the file in the most recent commit—all you need to do is git rm --cached <filename> and git commit --amend -C HEAD (same as the “most recent commit” steps)

Then to finish up, run git rebase --continue

## FAQs and Follow-ups
### Where is the example dataset from?
It contains earthquake data from the Nepal Earthquake Open Data Portal. Check out this cool GitHub repo with a machine learning analysis one of my students did, using this data:

### When I run git rebase -i it opens in VS Code, Atom, or Sublime Text, and Git acts like I’m closing the file immediately. What can I do?
Due to some quirks of how Git interacts with files, you’ll need to have your Git editor configured to be a command-line text editor rather than a richer editor like VS Code, Atom, or Sublime Text. I use Vim, which you can set as your editor by running this line in the terminal:
```
git config --global core.editor vim
```
You could also use Emacs if you prefer that interface:
```
git config --global core.editor emacs
```
Then you should be able to proceed with git rebase -i and edit the file and indicate which commit(s) you want to amend.

### Is an interactive rebase the only solution for removing large files like this?
No, the solution described here is not the only solution for this issue. Check out this StackOverflow post for several other approaches:

[How to remove/delete a large file from commit history in Git repository?](https://stackoverflow.com/a/2158271/11482491)
What you want to do is highly disruptive if you have published history to other developers. See "Recovering From…
stackoverflow.com

### What if I was trying to push the large file on purpose, since the other people working on the project need it?
First, try to see if you can make the file small enough. Maybe split it into a couple of files, remove parts of the data you don’t need, or compress it to make it smaller. If that doesn’t work, you’ll need to find some other way to distribute the file. If the file is fairly static, consider adding it to a cloud storage location, e.g. AWS S3, where you can usually get a decent amount of storage in the free tier. If you need version control on the file (i.e. to track changes), check out Git LFS, which is an open-source tool that typically costs more when you use a server like GitHub.

Can I use this same technique to “rewrite history” in other ways, e.g. removing a password from my Git history or adding the right author information? Yes, but you want to be very careful. With large files, GitHub prevents you from pushing your commits, so rewriting history in this way only affects code that is stored locally on your computer. For something like a password (or anything else that’s smaller than 100MB), GitHub doesn’t prevent you from pushing your commits, so it’s possible that other developers on your team have already pulled down your changes. You’ll need to use the --force flag to push, and your collaborators will need to follow these instructions. This approach should be the last resort, and it’s always better to avoid getting in this kind of situation in the first place.