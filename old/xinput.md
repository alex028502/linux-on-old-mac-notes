This is how I originally tried to fix my mouse clicking areas before I realised
that the Ubuntu MATE configuration was way ahead of me. This plan failed and
forced me to search a bit more when this setting kept getting undone, getting
overridden by the Ubuntu MATE settings system.  I tried adding a delay to the
startup script, but the main readme has a better plan now.

----

```
xinput set-prop bcm5974 "libinput Click Method Enabled" 0 1
```

The last two args are clicking areas off, and multi-finger on. They are allowed
to both be low, but they can't both be high.

To confirm that you have the same device you can go like this:

```
$ xinput list
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ bcm5974                                   id=11   [slave  pointer  (2)]
⎜   ↳ USB Optical Mouse                         id=14   [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
    ↳ Power Button                              id=6    [slave  keyboard (3)]
    ↳ Video Bus                                 id=7    [slave  keyboard (3)]
    ↳ Power Button                              id=8    [slave  keyboard (3)]
    ↳ Sleep Button                              id=9    [slave  keyboard (3)]
    ↳ Apple Inc. Apple Internal Keyboard / Trackpad     id=10   [slave  keyboard (3)]
    ↳ Apple Inc. Apple Keyboard                 id=12   [slave  keyboard (3)]
    ↳ Apple Inc. Apple Keyboard                 id=13   [slave  keyboard (3)]
```

It works if you put `11` instead of `bcm5974` except I don't know if it is
always `11`. When I wrote it down in my notes, it was `15` I think.

The entry that I am looking for is `bcm5974`. It says `id=11`.  It's `11` on
both my computers. I hope it never changes. In my notes it says `15`. I guess
I'll find out soon enough. If it changes, I'm gonna have to make a startup
script that finds the id in that list and then sets it, or find out a better
way to set it.

If after running that command, if you prefer that way this works, you can add
a startup script here:

![Start Up Applications in Control Centre](../images/startup.png)

Make the name and description whatever you want, and make the exact same
command you tried out.  To see if it worked, just log out and log back in.

You can see the current settings like this:

```
xinput list-props bcm5974
```

and also see what other settings are available.
