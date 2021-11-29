---
layout: post
title: Going crustaceous with my terminal.
excerpt: Using command line tools rewritten in rust.
date: 2021-04-09
updatedDate: 2021-04-13
tags:
    - post
    - rust
    - linux
    - cli
---

Okay! Okay! Okay! So not the most creative or original title out there but, hey iris wariris. Creativity ain't one of my strong suits. As a developer one of the most frequent questions you will hear upon encountering a new language will be, 'Well what is it used for?' quickly followed by 'Who uses this?'. This I assume must what going through a developers head as they go through the stack overflow survey and find out that rust is the most loved language. But wait. Didn't the language just hit 1.0 in 2015 ?. Yes! Is it really mature for production use you might ask. Yes. Five years is a really long time in developer years. Several well known companies use rust in production and others are soon set to add to the numbers. While there are many areas where rust is used, I will focus my attention today on command line tools to make the development experience for developers smooth.
<!-- more -->
## $ bat

![Using bat on my terminal to show git changes made to the.eleventy.js file](/going-crustaceous/bat-in-action.png)

<!-- ![Photograph of Dio.](/11r/uploads/bat-in-action.png 'Holy Diver') -->

It's a bird, its a plane, no its [bat](https://github.com/sharkdp/bat). On the Github page the developers describe bat as a cat (1) clone with wings (Get it... wings ... because ... ). Ever since I installed it I don't think I've ever thought about going back to cat, I mean why should I, when bat comes with syntax highlighting off the bat (pun intended), has git integration baked into it and you also have the option to pipe STDOUT and STDIN through it. What more could you ask for ? One thing I did not like is that it comes with a pager but do not fret, you can remove it by using `bat --paging=never` or the using the more convenient way and setting _BATPAGER to never_ in your _bashrc_ or _zshrc_ file.

```bash
#EXPORTING MY PATHS
export PATH="$PATH:/opt/netbeans/bin/"
export PATH="$PATH:/home/jim_4067/.cargo/bin"
export PATH="$PATH:/home/jim_4067/.bin/exercism"
export PATH="$PATH:/home/jim_4067/.local/bin"
export BAT_PAGER="" #THIS ONE
```

## $ dust

![running dust on the terminal to show it in use and how it show the apparent size of the files & folders](/going-crustaceous/dust-in-action.png)

"You live by the terminal, you die by the terminal", _wise man 1434_. That means you do everything on the terminal, you code? do it in the terminal. You want to listen to music? Do it on the terminal. You want to play games? Play cli games. Yes, total devotion till the day you kick it. This leads to the next tool dust. du + rust = dust (According to verified sources, the Math is correct). I an effort to easily see how his disk was being used up Andy created dust. Compared to Linux's dud this feel more... modern. It displays the space taken up in a directory from largest to smallest using bars to visually represent this. It should also noted that Apparent-size is calculated slightly different as shown in the pic above running dust.

## $ lsd

![using lsd to list the files in the working directory](/going-crustaceous/lsd-in-action.png)

[lsd](https://github.com/Peltoche/lsd) (not to be confused with the psychedelic) is a modern alternative to ls. On this entire list, it is the command I use the most on the terminal. On of the things that blew my mind away was that it came with icons to display the various files I list and also icons for folders. It makes finding items really easy without having to resort to grep. It also gets rid of the monotonous green that ls defaults to so big win for developer everywhere I guess. Depending on the configuration that you have made on your shell's resource configuration(rc) file you might want to add the snippet of code below. It just makes the colours... pop. (If you use this make sure you have the _lscolor-256color_ ASCII file).

```bash
if [[ ("$TERM" = *256color || "$TERM" = screen* || "$TERM" = xterm* ) && -f /etc/lscolor-256color ]]; then
    eval $(dircolors -b /etc/lscolor-256color)
else
    eval $(dircolors)
fi
```

## $ exa

![using exa to list the files in the working directory](/going-crustaceous/exa-in-action.png)

Next on our list of ls clones but written in another language that isn't C and offers splendid colours, we have [exa](https://github.com/ogham/exa). From the Github page, exa is a 'modern replacement for the venerable file-listing command line program ls'. To summarise 'exa attempts to be a more powerful, more friendly version of ls' as stated also from their Github page. It is yet another great tool from rust considering the 12.8k stars it also has on its Github. I am not fond of using it because lsd already exists but be sure to also give it a try.

## $ procs

![running procs to display the processes running](/going-crustaceous/procs-in-action.png)

[Procs](https://github.com/dalance/procs) is a replacement for ps that works on Linux and experimentally on macOS and Windows. Running the procs command without any arguments displays all processes running. The processes are displayed From the smallest to highest PID which is convenient. Of course there would be no point in rewriting ps without adding multi-coloured output, and it does this too.

## starship

![starship running in visual studio](/going-crustaceous/starship-in-action.png)

To round of this list of cli tools, We have starship, which is not actually a cli tool but a shell-prompt for astronauts (see the reason why it's included ). It provides an information rich layout for developers showing thing like the language you are predominantly working with, the git branch and many more. To install starship we first download the binary from the [starship site](https://starship.rs/) which has instructions for how to do this on all platforms and after the download is complete we add the startup script to the shell rc file. All of the steps are detailed very well in their site.

Starship is highly configurable, which is where it shines. You can change the prompt to suite your desired needs by making changes to your TOML file. For example you can choose whether you want the language you are working with to be shown, whether to show the your git branch and many more options which are detailed to great length [here](https://starship.rs/config/#prompt).

Of course there are many tools written in rust, some of them make command line life super convenient while others are there to just show that it can be done in rust just as it can on other languages. Rust might be a young language but it is gained a lot of usage in those years and only see it growing more and more.

Thank you and may the force be with you...
