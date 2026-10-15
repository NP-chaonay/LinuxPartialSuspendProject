# Config File (json)

```
{
	"regappcode": {
		"vscode":{ "allowapp":false, "usefreezeapproch":true, "doResumeOnWaylandC":true },
		"firefox":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"discord":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"kde_discover":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":false },
		"kde_cn":{ "allowapp":false, "usefreezeapproch":false,"doResumeOnWaylandC":false },
		"spotify":{ "allowapp":true, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"kde_pms":{ "allowapp":false, "usefreezeapproch":false,"doResumeOnWaylandC":true,"optattrib":{"perodicResume":[5,0.5]}  },
		"msedge":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"tdv":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true },
		"kde_kwin":{ "allowapp":false, "usefreezeapproch":true,"doResumeOnWaylandC":true }
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
kde_kwin=`find /sys/fs/cgroup/ | grep -i cgroup.freeze | grep kwin`

eval "echo \$$1 | sed 's|/cgroup\.freeze|/|g' | tr ' ' '\n'"
```
