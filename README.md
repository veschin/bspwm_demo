- [Specs](#orgf3ad6a5)
- [Pre-Reqs](#orgb046981)
- [Backgrounds and colors](#orge7818e4)
  - [`~/.config/bspwm/bspwmrc`](#org754e6e9)
- [Work layout](#org91f6e77)
  - [Binary layout](#orgc118c20)
  - [Real example](#org8c8c1d6)
- [Single monitor with 5 desktops](#org59de66e)
  - [`~/.config/bspwm/bspwmrc`](#orgb1d4d51)
- [Some rules for apps](#orgc05a71a)
  - [`~/.config/bspwm/bspwmrc`](#org70f5cb9)
- [Some hotkeys in sxhkd](#org1029519)
  - [`~/.config/sxhkd/sxhkdrc`](#orgec3549f)
- [Bspc hotkeys](#orga9badab)
  - [`~/.config/sxhkd/sxhkdrc`](#org3f60d4d)
- [Polybar examples](#org7b4999a)
  - [Top bar modules](#orgc774fe1)
  - [Module example](#org4d52287)
  - [Custom module](#org46a7210)
    - [Module](#org6b57ac9)
    - [Script](#org301db74)
    - [Result](#orga2d215b)



<a id="orgf3ad6a5"></a>

# Specs

![img](img/0_1.png)


<a id="orgb046981"></a>

# Pre-Reqs

-   Linux (Fedora36)
-   BSPWM - wm
-   Sxhkd - hotkeys
-   Polybar - status bar
-   Clojure/bb - general purpose/scripting lang
-   Bash/Fish - shell
-   Konsole - terminal
-   Wal & Feh - color palette from background & background setter


<a id="orge7818e4"></a>

# Backgrounds and colors

![img](img/1.png) ![img](img/2.png) ![img](img/3.png)


<a id="org754e6e9"></a>

## `~/.config/bspwm/bspwmrc`

```bash
wallpaper=`find ~/wallpapers/ | sort -R | head -1`
wal -i $wallpaper -n
feh --bg-fill $wallpaper --bg-fill $wallpaper
```


<a id="org91f6e77"></a>

# Work layout


<a id="orgc118c20"></a>

## Binary layout

![img](img/4.png)


<a id="org8c8c1d6"></a>

## Real example

![img](img/5.png)


<a id="org59de66e"></a>

# Single monitor with 5 desktops


<a id="orgb1d4d51"></a>

## `~/.config/bspwm/bspwmrc`

```bash
bspc monitor HDMI-A-0 -d 1 2 3 4 5
xrandr -s 1920x1080 -r 144

bspc config border_width         5
bspc config window_gap          15

bspc config split_ratio          0.52
bspc config borderless_monocle   true
bspc config gapless_monocle      true
```


<a id="orgc05a71a"></a>

# Some rules for apps


<a id="org70f5cb9"></a>

## `~/.config/bspwm/bspwmrc`

```bash
bspc rule -a Google-chrome desktop=3
bspc rule -a TelegramDesktop:telegram-desktop desktop=2
bspc rule -a Peek state=floating follow=on focus=on
```


<a id="org1029519"></a>

# Some hotkeys in sxhkd


<a id="orgec3549f"></a>

## `~/.config/sxhkd/sxhkdrc`

```bash
super + Return
	konsole

Print
	sudo kill -9 `pgrep flameshot` && \
	flameshot gui && \
	xclip -i `ls -d -r --no-icons $HOME/Pictures/* | head -1` \
	-selection clipboard \
	-target image/png

super + space
	rofi -show drun

alt + space
	rofi -show run
```


<a id="orga9badab"></a>

# Bspc hotkeys


<a id="org3f60d4d"></a>

## `~/.config/sxhkd/sxhkdrc`

```bash

# Move window
super + {Left,Down,Up,Right}
	bspc node -s {west,south,north,east}

# Cycle between windows
alt + {_,shift + }Tab
	bspc node -f {next,prev}.LOCAL

# Reload bspwm config
super + shift + r
	bspc wm -r && pkill -USR1 -x sxhkd

# Close window
super + w
	bspc node -c

# Move window to desktop
super + {_,alt +} {1-9}
	bspc {desktop -f,node -d} {1-9}
```


<a id="org7b4999a"></a>

# Polybar examples


<a id="orgc774fe1"></a>

## Top bar modules

```conf
modules-center = music-now
modules-left = xworkspaces xwindow
modules-right = vpn totp filesystem pulseaudio xkeyboard memory cpu; wlan eth date
```


<a id="org4d52287"></a>

## Module example

```conf
[module/cpu]
type = internal/cpu
interval = 2
format-prefix = "CPU "
format-prefix-foreground = ${colors.primary}
label = %percentage:2%%
```


<a id="org46a7210"></a>

## Custom module


<a id="org6b57ac9"></a>

### Module

Echo to polybar

```conf
[module/music-now]
type = custom/script
exec = "bb ~/scripts/src/media.clj"
exec-if = "playerctl status | grep Playing"
interval = 3
```


<a id="org301db74"></a>

### Script

```clojure
(ns media
  (:require [clojure.java.shell :as sh]
            [clojure.string :as string]))

(def ->int #(Long/parseLong %))
(def ->float #(Float/parseFloat %))

(defn ->out [& args]
  (-> (apply sh/sh args)
      (:out)
      (string/split #"\n")))

(defn crop [re-field re-crop & [parse?]]
  (->> (->out "playerctl" "metadata")
       (filter #(re-find re-field %))
       (mapv (comp (or parse? identity)
                  #(string/trim %)
                  #(string/replace % re-crop "")))))

(let [position (-> (->out "playerctl" "position")
                   (first)
                   (->float)
                   (int))
      length   (-> (crop #"length"
                         #"chromium mpris:\w+"
                         ->int)
                   (first)
                   (/ 1000000)
                   (int)
                   (#(/ % 100)))
      name     (crop #"artist|title|album"
                     #"chromium xesam:\w+")
      percent  (-> position
                   (/ length)
                   (int )
                   (str "%"))]

  (->> (conj name percent)
       (filter not-empty)
       (string/join " ~ ")
       (symbol)))
```


<a id="orga2d215b"></a>

### Result

`HORUS ~ Марс наш ~ 91%`