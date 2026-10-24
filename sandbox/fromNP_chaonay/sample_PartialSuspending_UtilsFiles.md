# Warning/General About these set of sample 
- For code of {Userside,AsRoot} ... Read How-To and Warning at head of file
  if your user's ID isnt 1000 then you have to configure the cgroup-finder manually about User ID (this is easy)
  
# Sample Config File (json) (for RootSide Version 4 +Userside Version7) 
- this code freeze many GUI app that seem to be CPU-hungry
- the limitation is that if GUI app is freezed but Wayland Compositor doesn't (at least in KWin), the GUI app crashed due to "Connected Reset By Peer" (which due to Wayland socket)
- so we freeze KWin too
- but freezing KWin ==> MPRIS system (system used for showing/control media player info in UI shell) hanged. ==> fixed by perodicResume
	- PS: it seem to be good... because KWin may have other DBus stuff to communicate from/to on... so perodicResume make KWin able to resolve these DBus ==> Which I think solve our MPRIS problem
- But perodicResume also wakes GUI app even screen is off (due to above  1st-mentioned limitation) ==> their audio shouldn't be heard while in this partial suspending? ==> fixed in Version7... thanks to temporary virtual audio output device lmao
- For EasyEffects User: congrats! we have found way to make this stuff work well with my system!... just freeze EasyEffects lmao. (or you can set EasyEffect not to force output device of app)
	- PS: oops I forget to tell you something ... what I meant of "we have found way to make this stuff work well with my system!" I mean
 		- In case want audio of freezed app not to be heard ... but that apps is using EasyEffects ==> it works
   		- In case want audio of some app to be heard ... but that apps is using EasyEffects ==> it didn't ... you have to bypass EasyEffects OR set EasyEffect not to force output device of app.... but trust me... if you want power saving... using something that able to render audio ahead for many seconds instead of realtime (such as mpv) that can gain much worthful power saving
     	- you can allow EasyEffects ... but any freezed apps that using EasyEffects ... you would heard that app's sound if the app is temporarily waken due to some special condition (such as doResumeOnWaylandC... which can be triggered by Wayland Compositor's Perodic Wakeup (set by tag: isWaylandComp) OR RootSide script (when user send input while screen is off)
   - PS2: if you choose to freeze EasyEffect instead of "set EasyEffect not to force output device of each app" ==> then you should set doResumeOnWaylandC to false ==> and do not open the window of EasyEffect before going screen off (to prevent crashing from 1st-mentioned limitation)
```
{
	"regappcode": {
		"vscode":{ "allowapp":false, "usefreezeapproch":true, "doResumeOnWaylandC":true },
		"firefox":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"discord":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"kde_discover":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":false },
		"kde_cn":{ "allowapp":false, "usefreezeapproch":false,"doResumeOnWaylandC":false },
		"spotify":{ "allowapp":true, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"kde_pms":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true,"optattrib":{"perodicResume":[5,0.5]}  },
		"msedge":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"tdv":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"kde_kwin":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true, "optattrib":{"isWaylandComp":true,"perodicResume":[0.25,0.0016666666666666668]} },
		"easyfx":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":false }
	}
}
```

# CGroup Finder (python)

```
#!/bin/bash
vscode=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | egrep "app-code|VScode|VSCode|vscode"`
firefox=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep -v \.mount | grep freeze | grep firefox `
discord=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep  app-discord-`
kde_discover=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep  kde.discover | grep notifier`
kde_cn=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep  kdeconnect`
spotify=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | egrep '(app-spotify_spotify@|snap.spotify.spotify)'`
kde_pms=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep plasmashell`
msedge="`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep microsoft | grep edge` `find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep chromium`"
tdv=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep freeze | grep tradingview`
kde_kwin=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep cgroup\.freeze | grep kwin`
easyfx=`find /sys/fs/cgroup/user.slice/user-1000.slice/ | grep cgroup\.freeze | grep easyeffect`

eval "echo \$$1 | sed 's|/cgroup\.freeze|/|g' | tr ' ' '\n'"

```
