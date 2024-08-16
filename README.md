# Custom waybar workspaces module for labwc



https://github.com/jenav/labwc-ws4waybar/assets/13712031/951acaa8-b801-4b58-b30a-c02cc428a355



It's a dirty little hack but it gets the job done. For it to work we need to compile labwc from source since we are adding 4 lines of code to get what we need (minus comments). It works as an indicator and as a swicher; we can either click each workspace number/indicator or use the mouse wheel.

### What you need installed?

- [wtype](https://github.com/atx/wtype): it provides the 'switcher' functionality part
- [inotify-tools](https://github.com/inotify-tools/inotify-tools): our watcher ([installation](https://github.com/inotify-tools/inotify-tools/wiki))

### How does it work?

We modify labwc to make it write the current workspace name to a file in /tmp (/tmp is mounted in RAM). We do so in the only place where labwc makes the actual workspace switch no matter the action taken by the user/environment. So it's simple and effective.

Then we just run a 'server' shell script that watches the aforementioned file for changes and reacts by letting the different waybar workspace modules (custom, one per workspace) know if they are the active one or not, so they can adjust accordingly, e.g., by changing it's appearance.

The workspace modules that we put in the bar are all one same shell script that receives its workspace number as a parameter when you define it in the waybar's 'config' file.

The communication between the 'server' and the 'modules' is done through named pipes (FIFO).

### The scripts

#### wc_labwc

This is the 'server' shell script that watches for changes in the file labwc writes to anytime there's a change of workspace (our hack) and proceed to send a message to each of our modules notifying them if they are the active one or not.

#### wc_module

Our actual modules for waybar. We instantiate as many of this script as workspaces we want. Each of them will read their state (active/inactive) from a different pipe.

#### wc_unique

Use this module in waybar if you want a unique module displaying the current active workspace number for example (second half of the video). When using this module you don't need to use the previous two, since it acts both as a 'server' (watcher) and 'client' (setter).

### What to do?

1. Modify the function `workspaces_switch_to()` in src/workspaces.c applying the patch `0000-expose-current-workspace.patch` or editing the file by hand. Compile and install.

	<details>
		<summary>Detailed information</summary>
		
		### General considerations
			
		I'll be using the release branch 0.7.4 of labwc as an example to avoid all kind of things that could go wrong.
		You can use a tty (alt+ctrl+F2...12) to run the steps or do it from another WM/DE.
	
		### Steps
		
		1 - Make sure you don't have labwc already installed via your package manager. If it is, uninstall it first.
		2 - Clone the labwc repo and switch to a release branch.
		
			$ git clone https://github.com/labwc/labwc
			$ cd labwc
			$ git checkout 0.7.4
		
		3 - Install the necessary dependencies for compilation and runtime (but don't compile it yet).
		
		Be aware of distro specific shenanigans. For example, in Arch, one must install wlroots version 0.17 which is called
		'wlroots0.17' and make sure you have only one version installed.
		
		Follow labwc's wiki for dependencies: https://github.com/labwc/labwc/wiki
		
		Aditional dependencies for us:
		- wtype: it provides the 'switcher' functionality part (https://github.com/atx/wtype)
		- inotify-tools: watching file's changes capability (https://github.com/inotify-tools/inotify-tools/wiki)
		
		4 - Modify the source code.
		
		You can either apply the patch '0000-expose-current-workspace.patch' from my repo:

		Copy the file 0000-expose-current-workspace.patch from my repo to the base folder of the labwc source code and
		apply the patch like this:

			$ patch -p 1 < 0000-expose-current-workspace.patch

		Or edit manually:

		Go into the 'src' directory in the labwc folder you just cloned.
		Edit the file named 'workspaces.c' with your favorite editor adding this lines from line 284 (this is specific to
		branch 0.7.4), inside the funcion 'void workspaces_switch_to(...)':
		
		    /* HACK */
		    FILE *fptr;
		    fptr = fopen("/tmp/labwc.current-ws", "w");
		    fputs(target->name, fptr);
		    fclose(fptr);
		    /*******/
		
		    ![20240807_09h20m33s_grim](https://github.com/user-attachments/assets/7d55731f-365a-4506-86f8-ea34c3360a47)
		
		
		Save the file and exit.
		
		5 - Compilation.
		
		On Arch, run this before compilation so it can find the wlroots libraries:
			$ export PKG_CONFIG_PATH='/usr/lib/wlroots0.17/pkgconfig'
		
		Compile:
			(standing in the labwc/ folder)
			$ meson build
			$ ninja -C build
		
		6 - Installation.
			$ meson install -C build
		
		By default the labwc binary is installed in the /usr/local/bin folder, so make sure you have it in your PATH
		environment variable:

			$ echo $PATH

		If it's missing you need to add this line at the end of your ~/.bashrc file (in case you use bash):

   			export PATH=$PATH:/usr/local/bin
   
		You should be able to run the modified 'labwc' from anywhere now.
		
	</details>

3. Download, **review** and copy the 3 shell scripts `ws_labwc` (server), `ws_module` and `ws_unique` to the `~/.config/waybar/scripts/` folder. Create the folder if it doesn't exist or adjust the paths to your liking. Make sure the scripts are executable.
	<details>
		<summary>Detailed information</summary>
		
		Clone the repo:
		$ git clone https://github.com/jenav/labwc-ws4waybar.git

		Create the scripts folder:
		$ mkdir ~/.config/waybar/scripts
	
		Copy the scripts to the destination:
		$ cp labwc-ws4waybar/ws_* ~/.config/waybar/scripts/

	</details>

4. Modify the workspaces names in the scripts `ws_labwc` and `ws_unique` to match the names defined in your labwc `rc.xml` config file (usually in `~/.config/labwc/rc.xml`).

	<details>
		<summary>Example config in ~/.config/labwc/rc.xml</summary>
		
		```xml
  		<!-- ... -->
		<desktops>
	 		<popupTime>1000</popupTime>
			<names>
	        		<name>Escritorio 1</name>
	        		<name>Escritorio 2</name>
	        		<name>Escritorio 3</name>
	        		<name>Escritorio 4</name>
			</names>
		</desktops>
		<!-- ... -->
		```
  
	</details>

5. Add the custom modules to your waybar config file (usually in `~/.config/waybar/config`). One per workspace. See my snipet below.
  **Adjust the wtype commands to match your labwc keymaps as defined in your labwc/rc.xml file**

I provide my snipets as an example:

<details>
<summary>file: ~/.config/waybar/config</summary>
	
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
</details>

<details>
<summary>file: ~/.config/waybar/style.css</summary>

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
</details>
