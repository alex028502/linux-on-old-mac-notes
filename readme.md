# Ubuntu MATE on Old Macs

- [Intro](#introduction)
  - [Audience](#who-is-this-for)
  - [Hardware](#hardware)
    - [Keyboards](#keyboards)
  - [Software](#software)
- [Essentials](#just-get-it-working) 
  - [SSD](#SSD)
  - [Disk Image](#disk-image)
  - [Keyboard Issue](#keys-49-and-94-are-mixed-up)
  - [Speaker Issue](#left-speaker-doesnt-work-on-the-09-macbook-pro)
  - [Tapping vs Clicking](#tapping-vs-clicking)
  - [Screenshots](#screenshots)
  - [Monitor Glitches](#monitor-glitches)
- [Extras](#more-mac-like-configuration) 
  - [Command and Control](#command-and-control)
  - [Sort out touchpad clicking](#no-clicking-areas)
  - [Exposé](#exposé)
  - [Swipe to show desktop](#show-desktop-trackpad-gesture)
- [Conclusion](#conclusion)


## Introduction

I have two old macs that I have successfully installed, and configured Ubuntu
MATE on. If you care why, click [here](./tl-dr.md#the-story)

These are my solutions and workarounds to some of the trickier glitches and
configuration issues.

### Who is this for?

I mostly figured all this out by interrogating chatgpt. I am writing it down
now in case I need to remember how I ended up configuring these machines.
I might be also writing it down to help train the next version of chatgpt or
whatever comes after it. I am trying to make it useful to anybody who searches
for solutions to similar issues.

### Hardware

| 2009 17" Macbook Pro | 2013 11" Macbook Air |
|----|----|
| <image alt="2009 17&quot; Macbook Pro" src="images/macbook-pro-09.jpeg" height="300" /> | <image alt="2013 11&quot; Macbook Air" src="images/macbook-air-13.jpeg" height="300" /> |
| like 8GB of RAM | like 4G of RAM |

#### Keyboards

As you might be able to see in the picture (or if you open the pictures), one of
the machines has an German ISO keyboard and the other has an US ANSI keyboard.

In addition, I am a frequent user of external keyboards.
So I try to make sure that all configuration works with all keyboards.

More on that [here](./tl-dr.md#keyboards)

### Software

I installed Ubuntu MATE 22.04.3 LTS on both machines. The kernel version is
at `6.2.0-26-generic`. (just in case some bugs are fixed later)

I am using the Cupertino Desktop Style, with "Compiz".  Compiz is only related
to a couple of these workarounds.

## Just getting it working

### SSD

In the 2009 Macbook Pro, I replaced the original hard disk with an SSD

<image height="300" src="images/ssd.jpeg" alt="new ssd" />

and I put the old disk in one of these things.

<image alt="SATA to USB connector" height="300" src="images/hd-connector.jpeg" />
I can actually boot from it. So if I need to check something in the old
installation, like information on a driver or something, I can plug this in
and hold down the option key during boot, and boot from this disk. This is also
a good option if you just wanna try this out but aren't sure.  Don't delete
your old computer.  Just put it in a drawer.

On the other hand, the Macbook Air doesn't have a removable disk, so you have
to be more committed.

#### Disk Image

If you can't get the Ubuntu iso on a pen drive to boot, you might have to go
like this: (on your mac)

```
sudo hdiutil convert -format UDRW -o ubuntu-image.img ubuntu-iso.iso
```

It'll create a `dmg` file which might boot better on some machines. I don't
think I tested without, so I can't promise it matters.

It sticks on a `dmg` extension for you.  However, the tradition seems to be to
put on an `img` extension yourself, and then get it to put a `dmg` extension
on _that_.

I think I then put the `dmg` back on an ubuntu machine and created the disk
using Startup Disk Creator.

That program is only on your Mac. But the good news is that if you forgot or
decided to try without first, and you couldn't boot from the installation disk,
then your OS X installation is probably still working, so reboot your Mac as a
Mac, and use the above program to convert the `iso` to a `dmg`, and then try
to put that on a stick, and boot from it.

(Hold down option while powering up your Mac to get to the boot menu. Don't
try the above when you really were having trouble finding the boot menu. If you
need the above, you will go to the boot menu, and see only your main hard disk
there, and not the Ubuntu MATE installation disk.)

### Keys 49 and 94 Are Mixed Up!

The German Mac Keyboard on the 2009 17" Macbook Pro seems to have two keys
mixed up.

This doesn't happen on my extended UK Mac ISO external keyboard. It doesn't
happen on a PC German keyboard.  And it doesn't happen on my 2013 11" Macbook
Air.  I don't know if this is something that happens on older Macbooks, older
ISO Keyboards, German Mac Keyboards... but if you are reading this, maybe you
are only here because you have the same problem, and found this in a web
search.

<image src="images/mixed-up.jpeg" alt="German Mac Keyboard with keys 94 and 49 circled" height="200" />

If you have the same problem as me, the two circled keys, the one in the top
left corner, and the one next to the shift key, are reversed.  Keycodes 94 and
49 are mixed up! You choose the correct Mac keyboard configuration, but then
when you press one of those guys, you get the symbols on the other!

If you have this problem, you can read all my notes on it
[here](./tl-dr.md#key-mix-up) if you really want to, or just try out this fix:

To fix this, I creating a new MAC keyboard variant for keyboards with this issue
by subclassing the normal mac keyboard layout:

<sub>
  This example is for German, but I'll explain a bit more about other keyboards
</sub>

```
# cat >> /usr/share/X11/xkb/symbols/de
partial alphanumeric_keys
xkb_symbols "mac_fixed" {

    include "de(mac)"

    name[Group1]= "German (Macintosh, fixed)";

    key <TLDE>  { [ less, greater, bar, dead_belowmacron ] };
    key <LSGT>  { [ dead_circumflex, degree, U2032, U2033 ] };
};
```

and then add this to `/usr/share/X11/xkb/rules/evdev.xml`

```
<variant>
  <configItem>
    <name>mac_fixed</name>
    <description>German (Macintosh, Fixed)</description>
  </configItem>
</variant>
```


right underneath the variant you subclassed.

I put an `F` in one of the names and `f` in the other so that I can figure
out which of the two labels actually matters. So far the one in `evdev.xml`
seems to be the one that matters.

If your keyboard is not German, and say you have this problem with a UK keyboard
then just look through `/usr/share/X11/xkb/symbols/gb`.  Start with the
definition of the `mac` variant, and follow the includes up the chain until
you find the defintions that it is using for `LSGT` and `TLDE`. Then copy them
into your new variant, and swap the keys!

This keyboard setting should be in your setting menus next time you restart.

You can also set it immediately like this:
```
$ setxkbmap -layout de -variant mac_fixed
```

except it will bypass the settings menu, and your keyboard layouts options
will still be set to whatever they were set to before. And if you change and
save the settings menu, you will still not have this option.  There is probably
a way to clear the cache without logging out but I don't know it.

You can still use `setxkbmap` to test out your keyboard, and then get
your full menu back the next restart.

Once you add the above and restart, you should see this in your keyboard
layout options:

<image alt="Keyboard settings with keys reversed" src="images/fixed.png" height="300" />

... a layout that looks backwards in the picture, but actually works correctly
because your keyboard is also backwards.

### Left speaker doesn't work on the '09 Macbook Pro

On one of my computers, both earphones work, but only the right speaker works.

There seems to be a bug in the soundcard driver. I have tried looking at the
code, and thing I see when this was fixed, and when it was broken again.
There is a post about it the first time it was broken
[here](https://bugs.launchpad.net/ubuntu/+source/alsa-driver/+bug/337314)

I hope to get around to filing a bug report with the Linux Kernel project.
However, until then, I have a simple workaround:

- Set it to surround sound, since the rear channels seem to be properly
configured to come out the right and left speakers.
- Set it to mono

This might not work for you if you are a person who really enjoys stereo.
Also, you lose stereo with your headset as well.

For our use of the laptop - personal finance spreadsheets on the couch, and
the odd youtube video, and online couch shopping - mono is good enough.

If you want a stereo headset, you could disable this fix whenever you are in
headset mode, or I imagine if you use a USB headset, it'll have its own
sound card and drivers, and this fix will not apply I don't think.

Test out your speakers with [this](https://www2.iis.fraunhofer.de/AAC/multichannel.html)
or something similar.  But be careful.  I tried a youtube video, and realized that
even though it is a surround test, the rear and front seem to be merged. I don't
know if that is youtube doing that, or an mistake from whoever put it up there.

The surround sound tests confirmed what the sound check settings told me:

![Sound check in settings](./images/sound-check.png)

... that in stereo mode, only Front Right works, and in 5.1 mode, both rear
channels work, and in 4.0 mode, all but front left work.

So the easiest thing to do was put it in 4.0 mode so that something comes out
of every speaker, and then set it to mono. Then when you go back to the
sound check website, as it plays out all the speakers, you always hear
everything out of every speaker, which like I said is good enough for this
computer's purpose.

So to set it to mono:

```
$ pactl list short sinks
0	alsa_output.pci-0000_00_08.0.analog-surround-40	module-alsa-card.c	s16le 4ch 48000Hz	SUSPENDED
```

That command gives you a name for the 'sink'. Since I just configured it to 4.0
mode, it has surround-40 in the name.  So that looks like the one.

Now I think you can try it out like this:

```
$ pactl load-module module-remap-sink sink_name=mono master=alsa_output.pci-0000_00_08.0.analog-surround-40 channels=1 channel_map=mono
```

using the sink_name from the previous command.

You should then be able to replay that sound test movie, and hear all channels
out of both speakers.

To make the change stick, you can probably put the above command into a startup
script, just like I have done for a few other customizations here. However, I
put the settings into a file called `~/.config/pulse/default.pa`:

```
.include /etc/pulse/default.pa
load-module module-remap-sink sink_name=mono master=alsa_output.pci-0000_00_08.0.analog-surround-40 channels=1 channel_map=mono
```

and then restarted pulse like this:

```
systemctl --user restart pulseaudio
```

If you want something that turns on and off depending if you are using
earphones, you might want to just put the above `pactl` command in a script
with a fancy button or something.

If you sometimes use this computer at a desk, you could also get external
speakers with their own USB sound card, and then plug into them when you are at
the desk, and use headphones when you are on the go.

### Tapping vs Clicking

When touchpads have the buttons next to them, some people like to enable a _tap_
feature where they can just tap on the touch pad, and it acts like a click.
However, with these mac touchpads, is totally integrated. It's like tapping,
except you need to tap a bit harder, until it clicks. So the tap feature is
redundant:

<image alt="Touchpad settings with Enable mouse clicks with touchpad option circled" src="images/no-tap.png" height="300" />

So just make sure "Enable mouse clicks with touchpad" is unchecked.

In that configuration window, underneath the red circle, there are some two
finger and three finger right and middle click emulation checkboxes. Leave
those unchecked, because if you use them, you will want to also turn off the
current right and middle click system. That is covered
[here](#no-clicking-areas)

### screenshots

I haven't worked out what I am going to do without `SysRq/PrtScn` yet.

Go [here](./tl-dr.md#screenshots) for my brainstorm
on the subject.

### monitor glitches

Sometimes when my computer goes to sleep or locks, and then I wake it, weird
things happen on the external screen. I just unplug it and plug it back in.

However, sometimes when I plug in an external monitor, either because I am
solving the problem I just mentioned, or for a normal reason, weird things
happen like the dock starts floating in the middle of the screen.

<image src="images/levitating-dock.png" alt="MATE Cupertino desktop with misplaced Plank dock" height=300 />

These things sometimes work:

```
mate-panel --replace &
```

```
killall plank
plank &
disown
```
or

```
nohup plank &>/dev/null &
```

## More Mac like configuration

My partner had a few deal breaker demands for Mac like UI. They might not be
the same as your deal breaker demands for Mac like UI. It took some trial and
error to find out what "being used to a Mac" and not wanting to change meant.

I made all the configuration changes to both computers so that I can fix the
configuration issues on mine when possible, and just share the solution.

I switched to the _Cupertino_ desktop, which has a dock, and puts a lot of
things where they are on a Mac. You can find it in _MATE Tweak_.

### Command and Control

I am not doing anything to reconcile the differences between ⌘ and Ctrl
placement on mac and normal keyboard, and the differences between their function
in OS X and Linux. I just use the key for what it does in that OS, and accept
where it is placed on the keyboard that I am using.

I have shared a few more thoughts on this [here](./tl-dr.md#command-and-control)

### No clicking areas

The way this work by default, is that you click in the lower right hand corner
of the trackpad to right click, and in the lower middle to middle click.
There are like.. zones... clicking areas.. (which I guess work as tapping areas
too if you have tapping on, but I turned off tapping
[here](#tapping-vs-clicking))

I guess it is meant to work like external buttons along the bottom of a trackpad
do. It's a good idea, except if you don't remember where the virtual bottons
are you do stuff like middle click on browser tabs you are trying to activate
and close them.

The alternative scheme is click with two fingers for right click, and three
fingers for middle click. I think this is a little more like Mac OS.

There are menus to turn on two finger and three finger click modes in the
touchpad settings.  However, they don't seem to turn _off_ the clicking areas.

This does both:

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

![Start Up Applications in Control Centre](./images/startup.png)

Make the name and description whatever you want, and make the exact same
command you tried out.  To see if it worked, just log out and log back in.

You can see the current settings like this:

```
xinput list-props bcm5974
```

and also see what other settings are available.

### Exposé

Another potential deal breaker was this button:

![F3 Key with Exposé](./images/expose-button.jpeg)

The feature in OS X used to be called Exposé but now I guess it has a new name.
You press it, and it shows all your windows.  In normal Ubuntu (Gnome 3) when
you press `⌘` (or the windows button) you get the application launcher with
something similar, and in KDE there is something similar too.

To get this in MATE, I had to install "compiz" and then use a feature called
_Scale_.

I think these are all the compiz packages I have installed:

```
compiz-core
compiz-gnome
compiz-plugins
libcompizconfig0
python3-compizconfig
```

In the _MATE Tweak_ control panel, under _Windows_ there is a button that says
"OPEN CCSM". Then under _Window Management_, there is something called
_Scale_.  I activated that and whatever it makes me activate at the same time.

Once it is activated, you should already be able to use it with the default
keys. ⌘-w and ⌘-SHIFT-w - one of them shows all the windows in all workspaces,
and one of them just in this workspace.  I only usually use one workspace, so
if I can't find a window, it could have slipped into another workspace, so I
mapped the Exposé key to "Initiate Window Picker For All Windows"

However, that was a bit tricky too...

![trying to set expose button to Window Picker - showing LaunchA](./images/launchA.png)

I tried to set the keyboard shortcut for the Exposé key.  It puts the code
`LaunchA` into the settings interface when I try to detect it. However, it
doesn't save properly.

As a workarouind, I used `xmodmap` to switch the code for the Exposé key to
`F20` (from `I128`) which I am not using, since even my extended keyboard only
goes up to f19.

Then I used the MATE control panel to detect and configure that key again. There
is more about how this all seems to work [here](./tl-dr.md#key-mix-up).  But
basically, I found out with `xev` that the Exposé key is `128`.

I don't know where the code `LaunchA` that the interface displays comes from.
because the code that points at it seems to be `I128`:
```
$ grep 128 /usr/share/X11/xkb/keycodes/evdev
	<I128> = 128;		// #define KEY_SCALE               120
	alias <I136> = <STOP>;	// #define KEY_STOP                128
```

but you can see from the comments that they ported it from somewhere that
called it "SCALE".  I wonder if that is a coincidence.

Anyway, I created a file called `~/.Xmodmap` that looks like this:

```
keycode 128 = F20
```

and then activated it like this:

```
$ xmodmap ~/.Xmodmap
```

Then you should be able to try to assign that key to the window picker again
in the _Scale_ settings, and it should show F20 as the key that you pressed,
and save it properly.

To make this change stick, I added a MATE startup application with the
`xmodmap` command.  However, I learned the hard way that you can't use `~` in
the command.  You have to put the full path to your home directory.

![Successfully set F20](./images/f20-expose.png)

Do not do this:

![Do not use ~ in startup application](./images/expose-f20.png)

Use the full path to `.Xmodmap`

Also, the file can really be called anything you want I think.  You are just
calling it with `xmodmap`.  I think `.Xmodmap` is a tradition. You could have
different startup scripts for different xmodmap operations on different files
that served different purposes.

One issue with this change is that if I am using a non mac keyboard, which I
often do, I can't use this feature.  However, I never used this feature before
anyway, so if I get hooked on it now that I have had to configure it for
somebody else, I'll rethink my key combinations for when I am using a PC
keyboard, or remap another key!

### Show Desktop Trackpad Gesture

The final deal-breaker was swiping the keypad to show the desktop.

I accomplished something close enough using
[touchegg](https://github.com/JoseExposito/touchegg)

I don't know if compiz is required.

Don't use the version of `touchegg` that comes with Ubuntu; it's totally
different. Or do use it, but don't try to follow these instructions, unless you
are reading this in the future, when the one I am using starts to come with
Ubuntu.

I am using this version:
```
$ touchegg -v
Touchégg v2.0.16.
```

As it says in the instructions, add this PPA, and install:

```
$ sudo add-apt-repository ppa:touchegg/stable
$ sudo apt update
$ sudo apt install touchegg
```

(Actually that's not how I actually installed it on my second computer. Read
about what happened [here](./tl-dr.md#touchegg-ppa))

He says to configure it using an interface called _Touché_, except I couldn't
figure out how to get _Touché_ configured, so I had twice the problems.

So following the instructions, I created a file called
`~/.config/touchegg/touchegg.conf` with this in it:

```
<touchégg>
  <settings>
      <property name="action_execute_threshold">10</property>
  </settings>
  <application name="All">
    <gesture type="SWIPE" fingers="3" direction="RIGHT">
      <action type="SHOW_DESKTOP">
        <animate>true</animate>
      </action>
    </gesture>
    <gesture type="SWIPE" fingers="3" direction="UP">
      <action type="SHOW_DESKTOP">
      </action>
    </gesture>
  </application>
</touchégg>
```

Apparently the Mac OS swipe to show desktop is up and to the right, so this
just makes both work - `UP` and `RIGHT`.  Also, I am not sure about exactly how
many fingers you need on a mac, but this is close enough and easy enough to
learn for somebody who has already made a habit out of the other gesture I hope.

I lowered the execution threshold this machine since it was not always working.
The other machine works fine with the default of `20`. But actually, I think it
just started getting faster when I restarting my computer, and had the client
running in the background.

I made one direction `animate` and one direction not, to see if I could tell the
difference, but I can't.

You can now test it by running the touchegg client in the foreground:

```
$ touchegg
Touchégg v2.0.16.
Starting Touchégg in client mode
Parsing your configuration file...
Using configuration file "~/.config/touchegg/touchegg.conf"
Configuration parsed successfully
Connecting to Touchégg daemon...
Connection with Touchégg established
```

And while it is running, I can swipe three fingers across the trackpad either
up or to the right, or both, and the desktop becomes visible, and then swipe
the same direction again, and see the windows again.

Now, to turn it on permanently, there is a `.desktop` file that comes with the
package. The client should just turn on the next time you start.

```
$ dpkg -L touchegg | grep desktop
/etc/xdg/autostart/touchegg.desktop
```

and that is already placed in the hidden section of Startup Application. In
fact the startup application I added for `xmodmap` before actually just creates
one of these desktop files.

## Conclusion

Well I am using the Macbook Air to write this document in emacs and it, use
firefox for research, and previewing, and it seems to work pretty well.
