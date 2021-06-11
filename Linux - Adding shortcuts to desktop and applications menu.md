# Adding shortcuts to desktop and applications menu

### To add a shortcut to the Applications menu:
```
cat > ~/.local/share/applications/eclipse.desktop

[Desktop Entry]
Version=1.0
Name=Eclipse
Exec=/opt/eclipse/eclipse
Terminal=false
Type=Application
StartupNotify=true
Categories=Development;IDE;
X-Desktop-File-Install-Version=0.23
```

### To add a shortcut to the Desktop:
```
cp ~/.local/share/applications/eclipse.desktop ~/Desktop/
```

>See https://developer.gnome.org/integration-guide/stable/desktop-files.html.en for more details

>Valid Categories: https://specifications.freedesktop.org/menu-spec/latest/apa.html

### What do these lines mean?
|line|description|
|---|---|
|`[Desktop Entry]`|The first line of every desktop file and the section header to identify the block of key value pairs associated with the desktop. Necessary for the desktop to recognise the file correctly.|
|Type=Application|Tells the desktop that this desktop file pertains to an application. Other valid values for this key are Link and Directory.|
|Encoding=UTF-8|Describes the encoding of the entries in this desktop file.|
|Name=Sample Application Name|Names of your application for the main menu and any launchers.|
|Comment=A sample application|Describes the application. Used as a tooltip.|
|Exec=application|The command that starts this application from a shell. It can have arguments.|
|Icon=application.png|The icon name associated with this application.|
|Terminal=false|Describes whether application should run in a terminal.|
