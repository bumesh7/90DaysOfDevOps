What to Review (pick at least one per section)

    Mindset & plan: revisit your Day 01 learning plan—are your goals still right? any tweaks?
    
    -> COncepts are getting harder need more time on hands-on and manage the office work, and still will not give up on tasks.
    
    Processes & services: rerun 2 commands from Day 04/05 (e.g., ps, systemctl status, journalctl -u <service>); jot what you observed today.

    $ systemctl status nginx
    $ journalctl -u nginx 10
    $ systemctl start nginx

    File skills: practice 3 quick ops from Days 06–11 (e.g., echo >>, chmod, chown, ls -l, cp, mkdir).

    $ cp test1.txt test2.txt
    $ mkdir test3 && cd test3
    $ ls -l
    $ echo "Hello" >> testing.txt
    
    Cheat sheet refresh: skim your Day 03 commands—highlight 5 you’d reach for first in an incident.

    $ top
    $ df -h
    $ free -h
    $ htp
    $ ps aux --sort=-%cpu
    
    User/group sanity: recreate one small scenario from Day 09 or Day 11 (create a user or change ownership) and verify with id/ls -l.

    $ sudo useradd -m tokyo -s /usr/bin/bash
    $ cat /etc/passwd | tail -4
    $ sudo addgroup developers
    $ sudo gpasswd -a toyo developers
    $ sudo -p /opt/dev
    $ sudo chown :developers /opt/dev

Mini Self-Check (write short answers in day-12-revision.md)

    Which 3 commands save you the most time right now, and why?

    $ pwd 
    $ uname -a
    $ ps
    $ top
    $ cat
    
    How do you check if a service is healthy? List the exact 2–3 commands you’d run first.

    $ systemctl status nginx
    $ journalctl -u nginx -n 20
    $ systemctl restart nginx

    How do you safely change ownership and permissions without breaking access? Give one example command.

    $ sudo chown -R ubuntu:devlopers /opt/dev
    $ sudo chmod -R 755 /opt/dev
    $ ls -l /opt/dev
    
    What will you focus on improving in the next 3 days?

    -> Learn Docker, Github and Github Actions, hands-on and Revision of previous topics.
