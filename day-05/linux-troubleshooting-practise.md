Run and record output for at least 8 commands (save snippets in your runbook)
Environment basics (2): uname -a, lsb_release -a (or cat /etc/os-release)
-> uname -a => Ubuntu kernel 6.17 running on x86_64. Kernel appears standard LTS.
<img width="1357" height="51" alt="image" src="https://github.com/user-attachments/assets/6a313fe1-9394-4dce-951f-7a468ed1f54b" />

-> lsb_release -a => Ubuntu 24.04 LTS – supported and stable release.
<img width="543" height="171" alt="image" src="https://github.com/user-attachments/assets/0b387104-b929-4fe1-8628-09caab7cb652" />

Filesystem sanity (2): create a throwaway folder and file, e.g., mkdir /tmp/runbook-demo, cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo

mkdir /tmp/runbook-demo
cp /etc/hosts /tmp/runbook-demo/hosts-copy
ls -l /tmp/runbook-demo

<img width="800" height="361" alt="image" src="https://github.com/user-attachments/assets/41fd3186-d314-44cf-8899-779bf42c2628" />

CPU / Memory (2): top/htop/ps -o pid,pcpu,pmem,comm -p <pid>, free -h, vm_stat (mac)

check sshd usage process => ps -o pid,pcpu,pmem

SSHD uses minimum cpu and memory.

<img width="721" height="135" alt="image" src="https://github.com/user-attachments/assets/977bdbd0-33e2-4923-80e1-58a89415a7de" />

Memory Overview

<img width="942" height="122" alt="image" src="https://github.com/user-attachments/assets/6b82903a-e83c-4b1c-bc6f-8e3de9b81573" />

Disk / IO (2): df -h, du -sh /var/log, iostat/vmstat/dstat

Disk usage => df -h

<img width="683" height="212" alt="image" src="https://github.com/user-attachments/assets/f9806173-95a4-48bb-b31b-558dfa23109e" />

Log Directory Size => du -sh /var/log   => Log directory size is normal

<img width="530" height="76" alt="image" src="https://github.com/user-attachments/assets/bceabdc9-e442-44cb-99b1-8d71a31a9cb9" />

Network (2): ss -tulpn/netstat -tulpn, curl -I <service-endpoint>/ping
ss -tulpn | grep ssh  => listen to port 22
curl -I http://localhost:22  => ssh is not http

<img width="630" height="106" alt="image" src="https://github.com/user-attachments/assets/1b7ccb9b-dcb7-4746-9357-91a11bba6c93" />

Logs (2): journalctl -u <service> -n 50, tail -n 50 /var/log/<file>.log

-> journalctl -u ssh -n 10
<img width="1912" height="298" alt="image" src="https://github.com/user-attachments/assets/f0dc350c-ca26-44e4-b630-4711740579cc" />

-> tsil -n 10 /var/log/auth.log
<img width="1587" height="278" alt="image" src="https://github.com/user-attachments/assets/86728484-ce8f-472a-81af-a3ded3f5ab39" />



Choose one target service/process (e.g., ssh, cron, docker, your web app) and stick to it for the drill.
For each command, add a 1–2 line note on what you observed (e.g., “CPU spikes to 80% when restarting”, “No recent errors in last 50 lines”).
End with a “If this worsens” section listing 3 next steps you would take (ex: restart strategy, increase log verbosity, collect strace).
Keep it concise and actionable (aim for ~1 page).

Restart the ssh
sudo systemctl restart ssh

Check the status of ssh
sudo systemctl status ssh

check if the ssh is listening to the port 22
ss -tulpn |grep :22

check for error logs
sudo journalctl -u ssh -n 10

validate ssh config syntax
sudo sshd -t
