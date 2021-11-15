# Others

- `stickybit`: enabling stickybit on a folder means, only user or root user can delete it, though other users have read permissions

- `dhcp process`: discover, offer, request, acknowlegement 

- `dmidecode`: For viewing system hardware components such as RAM(DIMMS), SMBIOS(system management BIOS), DMI--desktop management interface Table (not all reliable)

- `sudo ps -ef | grep <app>`: Select -e(every) process -f(full list) and grep out your desired application

- `sudo netstat -plunt | grep apache`: Find process running apache. Now obsolete. Use `ss`

- `top`: to check load average, swap, and which processes are using resources

- `zombie task`: Or `defunct` task. A process that has completed execution but still has an an entry in the process table. This is mostly child process where the entry is still needed to allow the parent process to read it's child exit status. The child process is dead but not yet been reaped. The kill process has no effect on zombie process. 

- `orphan process`: A process that is still running/executing but has its parent process dead. When the parent process dies, the orphaned process is adopted by the `init` process(ID 1)

- `steal time`: found in `sudo top` command

- `top`: press `M` to see process consuming memory

- `sudo df -hT`: Check disk filesystem

- `/var/log`: server logs

- `inode`: A uniquely existing number of all the files in a system. When a file is created, it's assigned an inode number and its name

- `journalctl -b`: View system logs