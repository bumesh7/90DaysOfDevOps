Follow these rules while creating your practice note:

Run and record output for at least 6 commands
Include 2 process commands (ps, top, pgrep, etc.)
<img width="1566" height="187" alt="image" src="https://github.com/user-attachments/assets/7b7ac303-6262-4434-8b2e-21378b8bae38" />

Include 2 service commands (systemctl status, systemctl list-units, etc.)
<img width="1456" height="442" alt="image" src="https://github.com/user-attachments/assets/5ccfeebd-b7e9-4eff-9fe4-5df82c730c28" />

Include 2 log commands (journalctl -u <service>, tail -n 50, etc.)
<img width="1471" height="331" alt="image" src="https://github.com/user-attachments/assets/236b318b-b9f6-4cb7-ab30-6a5987b07d6c" />

Pick one service on your system (example: ssh, cron, docker) and inspect it
Keep it simple and actionable
Suggested structure for linux-practice.md:

Process checks
Service checks
Log checks
Mini troubleshooting steps

1) Check Status = systemctl status ssh
   <img width="1878" height="656" alt="image" src="https://github.com/user-attachments/assets/17475bbd-c2e2-4009-b039-6a460ab58ac9" />

2) Start Service = sudo systemctl start ssh
3) Restart Service = sudo systemctl restart ssh
4) Check Configuration = sudo sshd -t
   
   Fix errors in /etc/ssh/sshd_config
