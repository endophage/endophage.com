+++
date = "2016-03-01T22:06:08-08:00"
title = "Docker Containers on the Desktop 2.0"

+++

Just over a year ago, having accepted an offer to work at Docker and on a short
break before starting, Jess Frazelle wrote 
[this awesome post](https://blog.jessfraz.com/post/docker-containers-on-the-desktop/)
on running your 
applications out of Docker containers. Jess talks about the benefits of the 
Apple App Sandbox and it sounds cool (it is), but Apple doesn’t give you the 
controls on Mac OS that they do on iOS, so an app can broadly define the
permissions it requires, not to mention people still install software from 
all over the internet.

You shouldn’t be trusting that software. If you haven’t read it already, you 
should add Future Crimes by Mark Goodman to your reading list. The first 
quarter of the book is a terrifying account of the unregulated data broker 
industry. We all know Facebook, Twitter, Google, etc.. aren’t really free, 
but that doesn’t mean we should give them free access to all the data on our 
personal computer. We can make it a good deal harder for their software to 
scrape information by using containers to isolate applications both from the 
host, and from eachother. 

So, using Jess’s post as a starting point, I wanted to create a more native 
experience of running an application in a container. What do I mean by this? 
I run Gnome 3 as my desktop environment, so I want:

- Not to have to futz with containers when I shut down the application and go to restart it later.
- A launcher in my Application menu, with a pretty icon.
- Not to have to give my user root level permissions (I’m that paranoid person that doesn’t give their user root perms and has to sudo docker ... all the time).

![](/images/spotify_container.jpg)

I’m going to use Spotify as my example app and we’ll tackle those 3 
requirements in order. First, let’s define a Dockerfile that will build our 
spotify image:

```
FROM ubuntu:latest

ENV HOME /home/spotify
ENV PULSE_SERVER unix:/pulse

RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys BBEBDCB318AD50EC6865090613B00F1FD2C19886 \
	&& echo deb http://repository.spotify.com stable non-free | sudo tee /etc/apt/sources.list.d/spotify.list \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends \
		spotify-client libpangoxft-1.0-0 libcanberra-gtk-module pulseaudio software-properties-common \
	&& rm -rf /var/lib/apt/lists/* \
	&& adduser --disabled-login --home $HOME --gecos "" spotify \
	&& adduser spotify audio \
	&& chown -R spotify:spotify $HOME

WORKDIR $HOME
USER spotify

CMD [ "spotify" ]
```

Build this Dockerfile and for the sake of having a concrete name throughout the
rest of this post, I’ll assume you tagged it as `spotify:latest`. Handling our 
first want is easy, we just need to add `--rm` to Jess’s version of the command. 
Note I’m also using pulseaudio, so I mount my host’s pulse socket to the 
location I defined as `PULSE_SERVER` in the Dockerfile above:

```
$ docker run --rm \
	-v /etc/localtime:/etc/localtime:ro \
	-v $HOME/.spotify:/home/spotify \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-v /run/user/$UID/pulse/native:/pulse \
	-e DISPLAY=unix$DISPLAY \
	--device /dev/snd:/dev/snd \
	--name spotify \
	spotify:latest
```

With --rm, when we close spotify (and kill it from the task bar), the container
is cleaned up, allowing us to cleanly relaunch the application. We also mount 
in a volume for spotify to use for the various bits of data it caches, this 
allows us to benefit from spotify’s caching and credential storage across 
restarts, again making it seem more native.

Now that we have our image and a container that cleans itself up, we want to 
add an application launcher for our container. While poorly documented, or at 
least, a little tricky to find an entry point for, adding things to the 
Application menu in Gnome is trivial. Gnome uses “Desktop Entries”, a 
freedesktop.org standard for defining how an application is launched, and 
how its menu entry looks.

Desktop entries live in two locations based on whether an application should 
appear for all users, or just for a specific user. System level entries that 
will appear for all users are defined at `/usr/share/applications`. User level 
entries that will appear only for a specific logged in user are defined at 
`~/.local/share/applications`. We will define a spotify.desktop file and 
place it in one of these locations (I put mine in `~/.local/share/applications`):

```
[Desktop Entry]
Type=Application
Name=Spotify
Icon=spotify
Exec="docker run --rm -v /etc/localtime:/etc/localtime:ro -v $HOME/.spotify:/home/spotify -v /tmp/.X11-unix:/tmp/.X11-unix -v /run/user/$UID/pulse/native:/pulse -e DISPLAY=unix$DISPLAY --device /dev/snd:/dev/snd --name spotify spotify:latest"
Terminal=false
```

This is a reasonably minimal desktop entry. Breaking it down, all desktop 
entry files start with `[Desktop Entry]`. The `Name` will be used as the 
application’s label in the menu. `Icon` defines the icon’s filename minus any 
file extension it might have. `Exec` is the command the be run when the 
application launcher is activated. Finally, `Terminal` defines whether the
application needs to be run in a terminal (spotify does not).

![Pretty icons are just a search away!](/images/spotify_icons.jpg)

The icons are defined beside the application locations at `/usr/share/icons` 
and `~/.local/share/icons` for the system and user level configuration. 
According to my testing an icon must be placed at the same level as the 
Desktop Entry file that uses it; i.e. a system level desktop entry means 
the icons must be placed in the system level icons folder. Within those 
icons folders you’ll find folders for different themes, which in turn 
contain folders for different sized icons, and finally, you’ll want to 
add your icon to the `apps` folder within each size folder. My (filtered) 
icons folder looks like this after adding my spotify icon in varying sizes:

```
~/.local/share $ tree icons 
icons
└── hicolor
    ├── 128x128
    │   └── apps
    │       └── spotify.png
    ├── 16x16
    │   └── apps
    │       └── spotify.png
    ├── 256x256
    │   └── apps
    │       └── spotify.png
    ├── 32x32
    │   └── apps
    │       └── spotify.png
    └── 48x48
        └── apps
            └── spotify.png
```

N.B. png is the best supported icon image format. You can use the `convert `
tool from imagemagick to change your icon from another image format via 
`convert input.ico output.png`

With our desktop entry and icons in place, the app should now show up in 
our application menu and be launchable (N.B. I had to restart X11 to make 
the icons register, not sure why). It’s time to deal with the final item 
on the list. The docker commands provided will have worked for you as is 
if you followed the docker docs and created a docker group, with root 
permissions and added your user to it. I don’t do that, I don’t trust
myself to not run some command, like `rm -rf /` when I’m not paying 
attention and break my system. More to the point, I don’t trust certain 
co-workers not to come up with some really sneaky ways to screw with my 
system in the ever escalating pony wars!

So, it breaks the “make it native” theme a little, but we can use a 
variety of tools to create an interative GUI version of `sudo`. On 
Gnome, you’ll want to use `gksu` (or `gksudo`). If you use `gksu`, you’ll 
need to configure it to use `sudo` under the hood (N.B. do not run this 
command as root):

```
$ gconftool-2 --set --type boolean /apps/gksu/sudo-mode true
```

We can just update the `Exec` command in our spotify desktop entry to add this 
command and we done! No need to restart anything on this one, just relaunch the
app and you’ll be greeted with a popup that asks for your password.

```
[Desktop Entry]
Type=Application
Name=Spotify
Icon=spotify
Exec=gksu "docker run --rm -v /etc/localtime:/etc/localtime:ro -v $HOME/.spotify:/home/spotify -v /tmp/.X11-unix:/tmp/.X11-unix -v /run/user/$UID/pulse/native:/pulse -e DISPLAY=unix$DISPLAY --device /dev/snd:/dev/snd --name spotify spotify:latest"
Terminal=false
```