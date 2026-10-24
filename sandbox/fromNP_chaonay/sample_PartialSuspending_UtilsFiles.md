# Sample Config File (json) (made on RootSide+Userside Version7) 
- this code freeze many GUI app that seem to be CPU-hungry
- the limitation is that if GUI app is freezed but Wayland Compositor doesn't (at least in KWin), the GUI app crashed due to "Connected Reset By Peer" (which due to Wayland socket)
- so we freeze KWin too
- but freezing KWin ==> MPRIS system (system used for showing/control media player info in UI shell) hanged. ==> fixed by perodicResume
- But perodicResume also wakes GUI app even screen is off (due to above  1st-mentioned limitation) ==> their audio shouldn't be heard while in this partial suspending? ==> fixed in Version7... thanks to temporary virtual audio output device lmao
- For EasyEffects User: congrats! we have found way to make this stuff work well with my system!... just freeze EasyEffects lmao. (or you can set EasyEffect not to force output device of app)
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
