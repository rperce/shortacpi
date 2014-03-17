shortacpi
=========

Parse the output of acpi down to something simple.

apci is great at reading battery data but bad at putting in a statusbar.

To download shortacpi, run

    wget https://raw.github.com/rperce/shortacpi/master/shortacpi
    chmod a+x shortacpi
    
and then move it to somewhere in your PATH, e.g.

    sudo mv shortacpi /usr/local/bin/
    


shortacpi works from any command-line that can see your PATH, or if you have a window manager that lets you create custom widgets of some sort (I use awesomewm), it's perfectly suited to live in a statusbar.


Adding to awesomewm
-------------------

(Below I make several references to `~/.config/awesome/`.  This will work for most users, but if the output of `echo $XDG_CONFIG_HOME` is neither empty nor `/home/<user>/.config/`, use that config directory instead.)

If, like me, you use [awesomewm](http://awesome.naquadah.org/), you can easily add this to your statusbar as a widget.  If you're not using a custom `rc.lua`, execute

    mkdir -p ~/.config/awesome
    cp /etc/xdg/awesome/rc.lua ~/.config/awesome/

Now you have a custom `rc.lua`.


Open the file `~/.config/awesome/rc.lua` in the editor of your choice.

In the file, search for `-- {{{ Wibox`; for me it was line 112.  Below that add the following:

    batterywidget = widget({ type = "textbox" })
    batterywidget.text = " | Battery | "
    batterytimer = timer({ timeout = 5 })
    batterytimer:add_signal("timeout", 
        function()
            fh = assert(io.popen("shortacpi", "r"))
            text = fh:read("*l*")
            if text ~= nil then
                batterywidget.text = " | "..text.." | "
            end
            fh:close()
        end
    )
    batterytimer:start()
    
On line two, `" | Battery | "` is whatever you want the widget to say before it polls acpi for battery status.  I like to separate my statusbar widgets with pipes, so I added those there.  If you don't like them, remove them from line 9 as well (`" | "..text.." | "`); the `..` syntax is the lua operator for string concatenation.

On line three, `timeout = 5` indicates that the display will update every five seconds.  Change the number to your liking.

---

Now search for `Create a promptbox for each screen`.  A few lines below that should be the line `mywibox[s].widgets = {`, followed by several things.  One of these should be `mylayoutbox[s]`; from that to the end of the group (the next `}`) is what shows up on the right side of your statusbar in right-to-left order.  My `rc.lua` has `mytextclock` as the next entry; I wanted my battery widget to the left of the clock, so on the following line I entered `batterywidget,`.  Place that line wherever you want, but make sure it is inside the group begun by the earlier line `mywibox[s].widgets = {`.

Once this edit is complete, exit the editor program and in a shell run `awesome -k`.  If the output is `✔ Configuration file syntax OK.`, you're done!  Restart awesome with, by default, `Mod4-Shift-R` and the battery widget should appear.  If it displays something different, something is wrong.  Email me or ask a forum or google for more information.
