---
title: Systemd
layout: default
---


Example with `kodi` :  `/lib/systemd/system/kodi.service`

```ini
[Unit]
Description = Kodi Media Center  
After = remote-fs.target network-online.target  
Wants = network-online.target  
  
[Service]  
User = pi
Group = pi
Type = simple
ExecStart = /usr/bin/kodi-standalone
; ExecStart = /usr/bin/su %i /usr/bin/kodi-standalone
Restart = on-abort  
RestartSec = 5  
  
[Install]  
WantedBy = multi-user.target
```

```
systemctl enable kodi
systemctl start kodi
```
