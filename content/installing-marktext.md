---
title: Installing marktext on Deepin or most Debian derived distros.
excerpt: Little guide on how to install marktext
date: 2021-11-28
updatedDate: 2021-11-28
tags:
- linux
---

So yeah, I messed with Linux yesterday and ended up paying the ultimate price. I was trying to delete the trash folder from the command line and ended deleting the local `bin` directory. Reading through a couple of the answers given to a poor soul who had done what I had done on Stack Overflow, I knew that I was going to reinstall the OS again. It wasn't the firs time I was was going to do this and it definitely that last. Most of my important documents included the 200GB homework folder, are kept in the data partition, so I wasn't worried about losing anything important to me. What I was worried about was reinstalling the software I regularly use, because let me tell. Anyone who has been unlucky enough to dabble in this Linux cult will tell you, that God has smiled on you if you actually installed and run a package on the first try. It's too rare an occasion that I think we should have a nobel prize for folks who have lady luck smiling on them when installing Linux.
<!-- more -->
Anyway with this new fresh start, while looking up an article trying to trouble shoot why I could not install typora using `add-apt-repository` (I'm on Deepin and most packages do not have distribution templates for this distro), I came across [marktext](https://marktext.app/). Do not get me wrong, I am in no way comparing the two MD editors together, I am just saying that after using it for a while, I like the look and feel of it. I mean look at it. Beautiful isn't it?

![](/installing-marktext/marktext-in-action.png)

Anyway here are the steps I followed installing the Editor on my system. Enjoy this rather quite long read which will have TL;DR at the end.

In installing marktext, you have two choices. As when this blog is being written there is no native package, so you either have to chose from from the flatpak or the appimage. I chose the appimage option which was really easy to install. Download the [appimage](https://github.com/marktext/marktext/releases/tag/v0.16.3) depending on your distro/OS. I am on Debian so here is how you install it after downloading the `marktext-amd64.deb` package. 

`sudo apt install ./marktext-amd64.deb`

Next we have to change the permission for the appimage using `chmod +x marktext-x86_64.AppImage` and make it executable. `chmod` is a command that is used to modify file permissions. These file permissions determine whether you can read, write, modify or execute a file. So with the above command we are saying that our file should be `executable` using the `x` flag and the `+`  flag defines whether the permissions can be added. You should now be able to run marktext using `./marktext-x86_64.AppImage` 

The next todo, is adding marktext to the launcher. First you will need to create a  `marktext.desktop` file in the `/usr/share/applications` directory. `sudo touch marktext.desktop` should do the trick. `.desktop` files contain meta information about a program in Linux. Generally, they contain the name, the icon and  where the program has be saved so that it can be run. Greater information can be found on the [arch wiki](https://wiki.archlinux.org/title/desktop_entries). Next depending on how deep you've been sucked into the Linux cult, using you favorite terminal editor, populate this *marktext.desktop* file using the contents below.

```bash
Name=Mark Text
Comment=Next generation markdown editor
Exec=/opt/marktext/marktext-x86_64.AppImage %F
Terminal=false
Type=Application
Icon=marktext
Categories=Office;TextEditor;Utility;
MimeType=text/markdown;
Keywords=marktext;
StartupWMClass=marktext
Actions=NewWindow;
[Desktop Action NewWindow]
Name=New Window
Exec=/opt/marktext/marktext-x86_64.AppImage --new-window %F
Icon=marktext
```

Now let's make marktext visible on the launcher. In the `/opt` directory, which stores optional software, added on by the user, `sudo mkdir /opt/marktext`. `sudo mv marktext-x86_64.AppImage /opt/app.marktext` into this directory and after a quick `update-desktop-database` you should have marktext installed on your system.

Here's the TL;DR promised

- download marktext appimage from the [releases page](https://github.com/marktext/marktext/releases/tag/v0.16.3)

- extract the downloaded package

- make it executable using `chmod +x marktext-x86_64.AppImage `

- create a `.desktop`` file as shown above and fill it out as per your desired preferences 
