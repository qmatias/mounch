# MOUNCH - a simple WOFI/ROFI launcher

## Description

Mounch is a simple launcher using [ROFI](https://github.com/davatorium/rofi) on Xorg or [WOFI](https://hg.sr.ht/~scoopta/wofi) on Wayland backed by a yaml configuration file.

You simply define a set of configuration blocks for some custom commands and it will show them with the awesome rofi. The items are sorted by frequency or as the order coming from the
`mounch.yaml` config file.

Why not just use [dmenu](https://frasertweedale.github.io/blog-redhat/posts/2020-12-01-openshift-crio-userns.html#creating-a-user-namespaced-pod---attempt-4) rofi module you may ask ?

* I simply did not want to have to create a `.desktop` file for every apps I have, and find it much simpler to have everything in  a simple yaml definition.
* I did not want to have the launcher offering everythign I have on my system
* I wanted a curated list
* I wanted some title that differs from what's shipped on the .desktop from the system.
* I switch often between desktop/wm (ie: gnome/sway) and display system (xorg/wayland) and I want entries depending of the current environement
* And I wanted to have static ordering as well as frequency based one, which is fitting my workflow.


## Requirements

* [rofi](https://github.com/davatorium/rofi) >= 1.6.0
* [wofi](https://hg.sr.ht/~scoopta/wofi) for wayland
* [PyYaml ](https://pypi.org/project/PyYAML/)

## Screenshot

![image](https://user-images.githubusercontent.com/98980/142888468-ce6a5f08-6c5d-496a-8b37-20f901c7ce3f.png)

## Installation

## Arch AUR

* Package available from here:

    https://aur.archlinux.org/packages/mounch/

  example config file in /usr/share/doc/mounch and helpers in /usr/share/mounch/

  rofi theme is preinstalled in /usr/share/rofi/themes/

## Manual

1. Checkout this repository with git :

        mkdir ~/git/
        git clone https://github.com/chmouel/mounch  ~/git/mounch

2. Copy the theme to your rofi config directory :

        mkdir -p ~/.config/rofi && cp ~/git/mounch/rofi/mounch.rasi ~/.config/rofi/

3. Setup your `mounch.yaml` in `~/.config/mounch/mounch.yaml` look at the example [here](./mounch.yaml).

4. Depending on how you setup your desktop setup a key to launch the [main script](./mounch.py)
   in `~/git/mounch/mounch.py` if you have followed thru.

5. Install [pyyaml](https://pypi.org/project/PyYAML/) python library (which you
   probably have already), packages are called

## Configuration

The basic definition look like this :

```yaml
firefox:
  binary: gtk-launch
  args: firefox
  description: "Firefox"
  icon: firefox
```

You are starting by a unique id as a key from this yaml snippet called firefox,
you launch a binary called
[gtk-launch](https://developer.gnome.org/gtk3/stable/gtk-launch.html) which is a
standard gtk tool who launch your desktop application, you give the args
`firefox` and you use the icon firefox which is picked up from your local icon
files.

* args can be a list, ie:
    args: `["firefox", "https://linux.com/"]`

* binaries are looked into your path.
* the icons is the standard freedesktop ones, add new ones in for example:

    `~/.local/share/icons/hicolor/scalable/apps/`

* and if you like you can add some condition too with the `if` keyword, it must return a python
  boolean, ie:

```yaml
firefox:
  binary: gtk-launch
  args: spotify
  description: "Spotify"
  icon: spotify
  if: 'shutil.which("spotify")'
```

see a most complete example [here](./mounch.yaml).

By default `mounch` add some sane arguments to `rofi` or `wofi` if you don't
want them and want to let the launcher config files set the options then use the
`-N/--no-defaults` option.

## Wayland

When detecting wayland it will try to use [wofi](https://hg.sr.ht/~scoopta/wofi) instead of rofi, you can specify --use-rofi to force use rofi.

There is currently a bug with the stable version of wofi where the
"--allow-images" flag doesn't get parsed.

The bug is fixed in the development branch of wofi and until then you will need
to add this directive directly to your `~/.config/wofi/config` :

```
allow_images=true
```

There is multiples options you can configure to tweak wofi with its config, see [wofi(5)](https://man.archlinux.org/man/wofi.5.en), here is mine :

```
allow_images=true
colors=colors
filter_rate=100
allow_markup=true
lines=15
```

It works well with the [`dracula`](https://github.com/dracula/wofi) wofi theme (as seen on the screenshot)

If you run [sway](https://swaywm.org/) you may want to do this for your configuration so sway execute the command directly :

```
set $menu mounch -p
bindsym $mod+d exec $menu|xargs swaymsg exec --
```

If you run with Gnome on Wayland, you may want to make it to center new window automatically or mutter (the gnome wm) will show the windows randomly around the screen, which is quite annoying with a launcher like wofi, you just need to run this command to make it automatically centers them :

```shell
dconf write /org/gnome/mutter/center-new-windows true
```

## Internals

We cache all entries in a simple file in $HOME/.cache/mounch/cache, it's simple and has only the id name and the number of times the entry has been launched.
Before launch we sort that list and display the top hit at the top and the rest as configured in the mounch.yaml config file.
After launch we increase the counter and rewrite the cache file.


## Misc

* **License**: [Apache License](./LICENSE)
* **Authors**: Chmouel Boudjnah <chmouel@chmouel.com>
