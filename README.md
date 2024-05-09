# Custom waybar workspaces module for labwc



https://github.com/jenav/labwc-ws4waybar/assets/13712031/951acaa8-b801-4b58-b30a-c02cc428a355



It's a dirty little hack but it gets the job done. For it to work we need to compile labwc from source since we are adding 4 lines of code to get what we need (minus comments). It works as an indicator and as a swicher; we can either click each workspace number/indicator or use the mouse wheel.

### What you need installed?

- [wtype](https://github.com/atx/wtype): it provides the 'switcher' functionality part
- [inotify-tools](https://github.com/inotify-tools/inotify-tools): owr watcher ([installation](https://github.com/inotify-tools/inotify-tools/wiki))

### How it works?

We modify labwc to make it write the current workspace name to a file in /tmp (/tmp is mounted in RAM). We do so in the only place where labwc makes the actual workspace switch no matter the action taken by the user/environment. So it's simple and effective.

Then we just run a 'server' shell script that watches the aforementioned file for changes and reacts by letting the different waybar workspace modules (custom, one per workspace) know if they are the active one or not, so they can adjust accordingly, e.g., by changing it's appearance.

The workspace modules that we put in the bar are all one same shell script that receives its workspace number as a parameter when you define it in the waybar's 'config' file.

The communication between the 'server' and the 'modules' is done through named pipes (FIFO).

### What to do?

- Modify the function `workspaces_switch_to()` in src/workspaces.c applying the patch `0000-expose-current-workspace.patch` or editing the file by hand.

- Download, **review** and copy the 3 shell scripts `ws_labwc` (server), `ws_module` and `ws_unique` to the `~/.config/waybar/scripts/` folder. Create the folder if it doesn't exist or adjust the paths to your liking. Make sure the scripts are executable.

- Adjust the workspaces names in `ws_labwc` and `ws_unique` to match their names as defined in the labwc `rc.xml` config file.

- Add the custom modules to your waybar config. One per workspace. **ADJUST wtype commands to match YOUR labwc KEYMAPS!**

I provide my snipets as an example:

> file: ~/.config/waybar/config

```jsonc
{
  //...
  "modules-left": ["custom/ws1", "custom/ws2", "custom/ws3", "custom/ws4"],
  // Alternatively:
  //"modules-left": ["custom/ws_unique"],
  //...
  //...
  "custom/ws_unique": {
  	"format": "{}",
  	"on-click": "wtype -M logo -P tab -m logo",
  	"on-click-right": "wtype -M logo -M shift -P tab -m logo -m shift",
  	"on-scroll-up": "wtype -M logo -M shift -P tab -m logo -m shift",
  	"on-scroll-down": "wtype -M logo -P tab -m logo",
  	"exec": "~/.config/waybar/scripts/ws_unique",
  	"exec-if": "test -f ~/.config/waybar/scripts/ws_unique",
  	"return-type": "json",
  	"tooltip": false
  },
  "custom/ws1": {
  	"format": "{}",
  	"on-click": "wtype -M logo -P 1 -m logo",
  	"on-scroll-up": "wtype -M logo -M shift -P tab -m logo -m shift",
  	"on-scroll-down": "wtype -M logo -P tab -m logo",
  	"exec": "~/.config/waybar/scripts/ws_module 1",
  	"exec-if": "test -f ~/.config/waybar/scripts/ws_module",
  	"return-type": "json",
  	"tooltip": false
  },
  "custom/ws2": {
  	"format": "{}",
  	"on-click": "wtype -M logo -P 2 -m logo",
  	"on-scroll-up": "wtype -M logo -M shift -P tab -m logo -m shift",
  	"on-scroll-down": "wtype -M logo -P tab -m logo",
  	"exec": "~/.config/waybar/scripts/ws_module 2",
  	"exec-if": "test -f ~/.config/waybar/scripts/ws_module",
  	"return-type": "json",
  	"tooltip": false
  },
  "custom/ws3": {
  	"format": "{}",
  	"on-click": "wtype -M logo -P 3 -m logo",
  	"on-scroll-up": "wtype -M logo -M shift -P tab -m logo -m shift",
  	"on-scroll-down": "wtype -M logo -P tab -m logo",
  	"exec": "~/.config/waybar/scripts/ws_module 3",
  	"exec-if": "test -f ~/.config/waybar/scripts/ws_module",
  	"return-type": "json",
  	"tooltip": false
  },
  "custom/ws4": {
  	"format": "{}",
  	"on-click": "wtype -M logo -P 4 -m logo",
  	"on-scroll-up": "wtype -M logo -M shift -P tab -m logo -m shift",
  	"on-scroll-down": "wtype -M logo -P tab -m logo",
  	"exec": "~/.config/waybar/scripts/ws_module 4",
  	"exec-if": "test -f ~/.config/waybar/scripts/ws_module",
  	"return-type": "json",
  	"tooltip": false
  },
}
```

> file: ~/.config/waybar/style.css

```css
/* ... */
#custom-ws_unique {
	padding: 0 6px;
	color: #cccccc;
	font-size: 15px;
}

#custom-ws1 {
	padding: 0 3px 0 6px;
	color: #707070;
}
#custom-ws2,
#custom-ws3 {
	padding: 0 3px 0 3px;
	color: #707070;
}
#custom-ws4 {
	padding: 0 6px 0 3px;
	color: #707070;
}

#custom-ws1.active,
#custom-ws2.active,
#custom-ws3.active,
#custom-ws4.active {
	color: #cccccc;
}
/* ... */
```
