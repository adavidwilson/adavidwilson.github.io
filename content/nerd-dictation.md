+++
title = "Nerd Dictation: Offline Dictation for Linux"
date = 2021-06-07
category = "Utilities"

[taxonomies]
categories = ["rice"]
tags = ["accessibility", "ricing", "configuration", "linux", "foss"]
+++

I've been suffering from a particularly bad bout of cubital tunnel syndrome recently due to less than ergonomic practices with my laptop and practicing classical guitar. This has led me to look into various accessibility solutions for typing with one hand. In this blog post, I'll be looking at the various solutions that I found. We'll start by going over how I found these solutions, what I ended up choosing (spoiler alert: nerd-dictation) then finally I'll talk about how I configured my system using sxhkd to toggle dictation by pressing the function key twice on my keyboard (hearkening back to my macOS days).
<!-- more -->
## My system

I must confess that I use Arch Linux which means that I have a passion for succumbing to configuration paralysis. This is a well documented condition where Arch users in particular spend days tweaking very small aesthetic minor features of their setup since nothing is given to you by default on a completed Arch install. All jokes aside, the benefits of using Arch are quite extensive. By default the installed system only has on the order of two hundred or so packages and uses very little resources (~40 MB of RAM in the TTY, and ~100 MB in X with dwm). Another confession I must make is that I love used ThinkPads. My current main machine is a T440p, so Arch's emphasis on minimalism helps keep my 7-year-old machine quick and responsive.

For my window manager, I use the Dynamic Window Manager (dwm) by suckless, and naturally I use st as my terminal. With all of this in mind let's go over how I discovered nerd-dictation and set it up on my system.

## Finding a solution to my accessibility problem

So the first instinct whenever you run into an issue on the computer, in my case this accessibility problem, is to check a search engine. I use DuckDuckGo just to avoid giving unnecessary information to Google. So to start, I went ahead and looked up one-handed typing accessibility solutions. The exact search query used was "one hand typing accessibility." The first link for me was to [sralab](https://www.sralab.org/lifecenter/resources/computer-access-options-one-handed-typing) and goes over the various options for one-handed typing.

It turns out that there are a number of different solutions to this problem. The first one that caught my eye was sticky keys. I had always assumed that this didn't really do anything and was just a way to annoy you if you were gaming in Windows. Despite the suggestion, I didn't think that this would offer much in the way of assisting my issue. Another solution stood out was the existence of a one-handed Dvorak keyboard layout. I use the standard Dvorak layout on all my machines, so this seemed like a possible solution, but I didn't really want to have to spend the time learning a completely new keyboard layout. Finally the last solution that stood out to me was dictation. Once I decided on going down that path, I checked my next resource&mdash;the Arch Linux wiki.

## The Arch wiki

If you've ever used Arch Linux before, you know that the [wiki](https://wiki.archlinux.org) is an indispensable resource for being able to do pretty much anything. Whenever I'm looking for new software utilities the first place that I check is the applications list. For this particular case there's actually a [section dedicated to speech recognition](https://wiki.archlinux.org/title/List_of_applications/Other#Speech_recognition). After quickly looking through the GitHub repositories of all of the different projects listed, I decided on going with nerd-dictation since it seemed to be the most recently updated and was specifically created to allow for dictation-based typing.

## Nerd Dictation

So what is [nerd-dictation](https://github.com/ideasman42/nerd-dictation) anyway? In the words of its creator it's a tool for "offline speech to text for desktop Linux." It's simple, hackable, and has zero overhead (since activation is fully manual). A quick look through the readme reveals that it is a python script that uses the VOSK-API and has a few creature comfort features. All you need to do is install the vosk package using pip, clone the repository, put the executable on the path somewhere, download a language model, and then you're off to the races.

In my case I also wanted a simple way to toggle dictation, so I ended up using the simple x hotkey daemon (`sxhkd`) to set up a key bind for a custom script that checks for the existence of a file and toggles whether or not under dictation is running. 

## Keybind setup and use

Here's the toggle script for starting and stopping dictation. It checks to see if a file is present in my user's cache directory. If it isn't, it starts dictation and creates the file also sending out of a notification. Then when the key bind is pressed again, it checks to see if the file exists once more. If it does, it stops dictation, removes the file and finally sends out another notification:
```fish
#!/usr/bin/env fish

if test -e $XDG_CACHE_HOME/nerd-dictation
  nerd-dictation end --cookie $XDG_CACHE_HOME/nerd-dictation
  rm $XDG_CACHE_HOME/nerd-dictation
  notify-send "Dictation" "Dictation stopped"
else
  notify-send "Dictation" "Dictation started"
  nerd-dictation begin --cookie $XDG_CACHE_HOME/nerd-dictation --full-sentence
end
```

The actual hotkey setup is simple using `sxhkd`. Just put the following in your `sxhkdrc` making sure `toggle-dictation` is on your `PATH`:
```fish
XF86WakeUp ; XF86WakeUp
  toggle-dictation
```

## Conclusion

I hope this blog post helps anyone with similar issues and provides some information on how to configure your own Linux system whenever you run into issues like this which usually require more esoteric utilities.
