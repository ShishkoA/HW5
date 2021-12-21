## Processes
```bash
1. Run a sleep command three times at different intervals
[root@localhost linux]# sleep 10000 &
[1] 1557
[root@localhost linux]# sleep 11000 &
[2] 1558
[root@localhost linux]# sleep 12000 &
[3] 1559
```
2. Send a SIGSTOP signal to all of them in three different ways.
```bash
[root@localhost linux]# kill -SIGSTOP 1557

[1]+  Stopped                 sleep 10000
[root@localhost linux]# kill -SIGTSTP 1558

[2]+  Stopped                 sleep 11000
[root@localhost linux]# kill -SIGTTIN 1559

[3]+  Stopped                 sleep 12000
```
3. Check their statuses with a job command
```bash
[root@localhost linux]# jobs
[1]   Stopped                 sleep 10000
[2]-  Stopped                 sleep 11000
[3]+  Stopped                 sleep 12000
```
4. Terminate one of them. (Any)
```bash
kill -SIGTERM 1557
[root@localhost linux]# jobs
[2]-  Stopped                 sleep 11000
[3]+  Stopped                 sleep 12000
```
5. To other send a SIGCONT in two different ways.
```bash
[root@localhost linux]# kill -SIGCONT 1558
[root@localhost linux]# jobs
[2]-  Running                 sleep 11000 &
[3]+  Stopped                 sleep 12000

[root@localhost linux]# kill -18 1559
[root@localhost linux]# jobs
[2]-  Running                 sleep 11000 &
[3]+  Running                 sleep 12000 &
```
6. Kill one by PID and the second one by job ID
```bash
[root@localhost linux]# kill -9 1558
[root@localhost linux]# jobs
[2]-  Killed                  sleep 11000
[3]+  Running                 sleep 12000 &
[root@localhost linux]# kill %3
[root@localhost linux]# jobs
[3]+  Terminated              sleep 12000
```

## systemd
1. Write two daemons: one should be a simple daemon and do sleep 10 after a start and then do echo 1 > /tmp/homework, the second one should be oneshot and do echo 2 > /tmp/homework without any sleep
```bash
[root@localhost linux]# cat /usr/local/bin/daem1.sh
#!/bin/bash

sleep 10 && echo 1 > /tmp/homework

[root@localhost linux]# cat /etc/systemd/system/mysleep.service
[Unit]
Description = I like sleeping service

[Service]
RemainAfterExit=true
ExecStop=/usr/local/bin/daem1.sh

[Install]
WantedBy=multi-user.target

[root@localhost linux]# cat /usr/local/bin/daem2.sh
#!/bin/bash

echo 2 > /tmp/homework


[root@localhost linux]# cat /etc/systemd/system/mysleep2.service
[Unit]
Description = I like output in single file /tmp/homework
After=mysleep.service
Wants=mysleep2.timer

[Service]
ExecStart=/usr/local/bin/daem2.sh
Type=oneshot

[Install]
WantedBy=multi-user.target

```
2. Make the second depended on the first one (should start only after the first)
```bash
After=mysleep.service
```
3. Write a timer for the second one and configure it to run on 01.01.2019 at 00:00
```bash
[root@localhost linux]# cat /etc/systemd/system/mysleep2.timer
[Unit]
Description=Execute at 00:00 on 01.01.2019

[Timer]
OnCalendar=2019-01-01 00:00:00
Unit=mysleep2.service

[Install]
WantedBy=multi-user.target
```
4. Start all daemons and timer, check their statuses, timer list and /tmp/homework
```bash
[root@localhost linux]# systemctl status mysleep
● mysleep.service - I like sleeping service
   Loaded: loaded (/etc/systemd/system/mysleep.service; enabled; vendor preset: disabled)
   Active: active (exited) since Sat 2021-12-18 13:17:27 MSK; 2h 20min ago

Dec 18 13:17:27 localhost.localdomain systemd[1]: Started I like sleeping service.
[root@localhost linux]# systemctl status mysleep2
● mysleep2.service - I like output in single file /tmp/homework
   Loaded: loaded (/etc/systemd/system/mysleep2.service; disabled; vendor preset: disabled)
   Active: inactive (dead) since Sat 2021-12-18 15:25:29 MSK; 13min ago
 Main PID: 2363 (code=exited, status=0/SUCCESS)

[root@localhost linux]# systemctl status mysleep2.timer
● mysleep2.timer - Execute at 00:00 on 01.01.2019
   Loaded: loaded (/etc/systemd/system/mysleep2.timer; enabled; vendor preset: disabled)
   Active: active (elapsed) since Sat 2021-12-18 15:26:23 MSK; 15min ago

Dec 18 15:26:23 localhost.localdomain systemd[1]: Stopped Execute at 00:00 on 01.01.2019.
Dec 18 15:26:23 localhost.localdomain systemd[1]: Stopping Execute at 00:00 on 01.01.2019.
Dec 18 15:26:23 localhost.localdomain systemd[1]: Started Execute at 00:00 on 01.01.2019.
[root@localhost linux]# cat /tmp/homework
2
```

5. Stop all daemons and timer
systemctl stop mysleep2.timer
systemctl stop mysleep2
systemctl stop mysleep

## cron/anacron
1. Create an anacron job which executes a script with echo Hello > /opt/hello and runs every 2 days
```bash
[root@localhost linux]# cat /etc/anacrontab | tail -1 
2       5       two_daily               echo Hello > /opt/hello
```
2. Create a cron job which executes the same command (will be better to create a script for this) and runs it in 1 minute after system boot.
```bash
crontab -e
@reboot sleep 60 && /home/linux/script.sh
chmod +x  /home/linux/script.sh
```
3. Restart your virtual machine and check previous job proper execution
```bash
[root@localhost linux]# reboot
Using username "linux".
Authenticating with public key "rsa-key-20211006" from agent
Last login: Sat Dec 18 11:03:05 2021 from 192.168.0.31
[linux@localhost ~]$ sudo su
[sudo] password for linux:
[root@localhost linux]# ls /opt
[root@localhost linux]# ls /opt
[root@localhost linux]# ls /opt
[root@localhost linux]# ls /opt
[root@localhost linux]# ls /opt
hello
[root@localhost linux]# ls /opt
hello
[root@localhost linux]#
```

## lsof
1. Run a sleep command, redirect stdout and stderr into two different files (both of them will be empty).
```bash
sleep 10000 & > output.txt 2> error.txt
```
2. Find with the lsof command which files this process uses, also find from which file it gain stdin.
```bash
[root@localhost linux]# lsof -p 1576
COMMAND  PID USER   FD   TYPE DEVICE  SIZE/OFF     NODE NAME
sleep   1576 root  cwd    DIR  253,0      4096 33625698 /home/linux
sleep   1576 root  rtd    DIR  253,0       250       64 /
sleep   1576 root  txt    REG  253,0     33128 50441322 /usr/bin/sleep
sleep   1576 root  mem    REG  253,0 106176928 50333147 /usr/lib/locale/locale-archive
sleep   1576 root  mem    REG  253,0   2156592     8812 /usr/lib64/libc-2.17.so
sleep   1576 root  mem    REG  253,0    163312  1020088 /usr/lib64/ld-2.17.so
sleep   1576 root    0u   CHR  136,0       0t0        3 /dev/pts/0
sleep   1576 root    1u   CHR  136,0       0t0        3 /dev/pts/0
sleep   1576 root    2u   CHR  136,0       0t0        3 /dev/pts/0

[root@localhost linux]# lsof -a -p 1576 -d 0
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sleep   1576 root    0u   CHR  136,0      0t0    3 /dev/pts/0
```
3. List all ESTABLISHED TCP connections ONLY with lsof
```bash
[root@localhost linux]# lsof -nP -iTCP -sTCP:ESTABLISHED
COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1522  root    3u  IPv4  21027      0t0  TCP 192.168.0.133:22->192.168.0.31:52425 (ESTABLISHED)
sshd    1525 linux    3u  IPv4  21027      0t0  TCP 192.168.0.133:22->192.168.0.31:52425 (ESTABLISHED)
```
