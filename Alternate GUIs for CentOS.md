# Alternate GUIs for CentOS

### Step 1 – Install xRDP on CentOS 7
Update your package index:
```
$ sudo yum -y update
```
Install xRPD (see also Remote Desktop to a CentOS machine)
```
$ sudo yum install -y epel-release
$ sudo yum install -y xrdp
$ sudo systemctl enable xrdp
$ sudo systemctl start xrdp
```
Open port `3389/tcp` for RDP in the firewall
```
$ sudo firewall-cmd --add-port=3389/tcp --permanent
$ sudo firewall-cmd --reload
```
### Step 2 – Install Your Preferred Desktop Environment
#### XFCE Desktop Environment
To install XFCE, run the following commands:
```
$ sudo yum install -y epel-release
$ sudo yum groupinstall -y "Xfce"
$ sudo reboot
```
Create the `.Xclients` file in the directory of the user you’re connecting with:
```
$ echo "xfce4-session" > ~/.Xclients
$ chmod a+x ~/.Xclients
```
May be needed?
```
echo "exec /usr/bin/xfce4-session" >> ~/.xinitrc
```
#### Uninstalling XFCE
To uninstall XFCE, run the following commands:
```
$ sudo yum groupremove -y "Xfce"
$ sudo yum remove -y libxfce4*
```

#### MATE Desktop Environment
To install MATE, run the following commands:
```
$ sudo yum install -y epel-release
$ sudo yum groupinstall -y "MATE Desktop"
$ sudo reboot
```
Create the `.Xclients` file in the directory of the user you’re connecting with:
```
$ echo "mate-session" > ~/.Xclients
$ chmod a+x ~/.Xclients
```
May be needed?
```
echo "exec /usr/bin/mate-session" >> ~/.xinitrc
```
#### Uninstalling MATE
To uninstall MATE, run the following commands:
```
$ sudo yum groupremove -y "MATE Desktop"
$ sudo yum autoremove -y
```
#### GNOME Desktop Evironment
To install GNOME, run the following commands:
```
$ sudo yum groupinstall "GNOME DESKTOP" -y
```
>This may take a while. There were ~1000 packages installed on a minimal CentOS 7 installation

##### Start the GUI

Although we installed the GNOME Desktop package group, the GUI will not be loaded by default on reboot.

We can check this by running:
```
$ systemctl get-default
multi-user.target
```
If our default target is `multi-user.target`, it means that the GUI will not be loaded. What we want is to set the default target to `graphical.target`.

To do this, run the following commands:
```
$ sudo systemctl set-default graphical.target

Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/graphical.target.
```
After which, run the following command to change to the GUI immediately:
```
$ sudo systemctl isolate graphical.target
```
Maybe needed?
```
echo “exec gnome-session” >> ~/.xinitrc
```
#### Uninstalling GNOME
To uninstall GNOME, run the following commands:
```
$ sudo yum groupremove -y "GNOME Desktop"
$ sudo yum autoremove -y
```