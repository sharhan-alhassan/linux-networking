
## 
- Linux run levesl 0 to 6
- ps -ef | grep init            # init has 1 because it's the first process to run
- Run levels in `init` is now being replaced with `target` in Ubuntu >=16.04 

```bash
Mapping between runlevels and systemd targets
   ┌─────────┬───────────────────┐
   │Runlevel │ Target            │
   ├─────────┼───────────────────┤
   │0        │ poweroff.target   │
   ├─────────┼───────────────────┤
   │1        │ rescue.target     │ 
   ├─────────┼───────────────────┤
   │2, 3, 4  │ multi-user.target │
   ├─────────┼───────────────────┤
   │5        │ graphical.target  │
   ├─────────┼───────────────────┤
   │6        │ reboot.target     │
   └─────────┴───────────────────┘

   # 1. singler user(root). No daemons started
   # 2. multi-user. No daemons or network interfaces
   # 3. multi-user. NO GUI
   # 4. Undefined. Can be defined by a user
   # 
```

- Runlevels directory: `/etc/rc0-6.d/`

## Systemd
```bash
systemctl daemon-reload         # reload all services
systemctl list-unit-files       # see all service enabled/disabled
systemctl get-default           # Get current target(runlevel)
systemctl set-default graphical.target  # Set target(runlevel) to graphical(or deprecated runlevel 5)
```
