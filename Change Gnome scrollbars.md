# Change Gnome scrollbars

>https://askubuntu.com/questions/775201/how-do-i-get-a-bigger-static-scrollbar-aka-normal-scrollbar#answer-871075

Here are the steps to restore the scrollbars which don't disappear and have permanent width, i.e. "classic" ones.

Tested in GNOME Flashback session in Ubuntu 16.04 (works in Unity, too).

1. Make a backup of `/etc/environment`. Run `sudo vi /etc/environment` and add there the next line:
    ```
    GTK_OVERLAY_SCROLLING=0
    ```
    This will prevent scrollbar's autohide behavior.

1. In order to avoid tampering with the main theme file `/usr/share/themes/Ambiance/gtk-3.0/gtk-widgets.css` we'll borrow some code from there, alter it and put it into user profile folder.  
    Create `~/.config/gtk-3.0/gtk.css` and put the next lines into it:
    ```
    /* Adding the buttons on the edges (if you don't need them, skip the next 4 lines)
    */
    
    .scrollbar {
    -GtkScrollbar-has-backward-stepper: 1;
    -GtkScrollbar-has-forward-stepper: 1;
    }
    
    /* Scrollbar trough squeezes when cursor hovers over it. Disabling that
    */
    
    .scrollbar.vertical:hover:dir(ltr),
    .scrollbar.vertical.dragging:dir(ltr) {
    margin-left: 0px;
    }
    
    .scrollbar.vertical:hover:dir(rtl),
    .scrollbar.vertical.dragging:dir(rtl) {
    margin-right: 0px;
    }
    
    .scrollbar.horizontal:hover,
    .scrollbar.horizontal.dragging,
    .scrollbar.horizontal.slider:hover,
    .scrollbar.horizontal.slider.dragging {
    margin-top: 0px;
    }
    
    /* Slider widens to fill the scrollbar when cursor hovers over it. Making it permanent
    */
    
    .scrollbar.slider.vertical:dir(ltr):not(:hover):not(.dragging) {
    margin-left: 0px;
    }
    
    .scrollbar.slider.vertical:dir(rtl):not(:hover):not(.dragging) {
    margin-right: 0px;
    }
    
    .scrollbar.slider.horizontal:not(:hover):not(.dragging) {
    margin-top: 0px;
    }
    ```
1. Create `~/.config/gtk-3.0/settings.ini` and add the next lines into it:
    ```
    [Settings]
    gtk-primary-button-warps-slider = false
    ```
    This will restore page-by-page scrolling behavior when you click the scrollbar on the either side of the slider. If this file already exists, just add the last line in the `[Settings]` section of it.
1. Uninstall `overlay-scrollbar` and `overlay-scrollbar-gtk2` packages - you won't need them anymore.
    >If you use some applications which need superuser rights then you should also place `gtk.css` and `settings.ini` files into the root's profile folder:
    ```
    sudo cp ~/.config/gtk-3.0/gtk.css /root/.config/gtk-3.0/
    sudo cp ~/.config/gtk-3.0/settings.ini /root/.config/gtk-3.0/
    ```
    >If you find that these scrollbars are too narrow for you, make them broader. Just add the next line to your `~/.config/gtk-3.0/gtk.css`:
    ```
    .scrollbar {
        -GtkRange-slider-width: 15;
    }
    ```
    Increase the width as you see fit (the default is 10). Update `/root/.config/gtk-3.0/gtk.css` also, if needed.
