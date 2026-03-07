Quick Concepts (write 1–2 bullets each)

    OSI layers (L1–L7) vs TCP/IP stack (Link, Internet, Transport, Application)
```
L7-> Application   \  
L6-> Presentation    > Application
L5-> Session       /  
L4-> Transport      -> Transport
L3-> Network        -> Internet
L2-> Data Link      \ 
L1-> Physical       -> Link

Where IP, TCP/UDP, HTTP/HTTPS, DNS sit in the stack

HTTP / HTTPS	     Application -> Web communication
DNS	               Application ->	Domain → IP resolution
TCP	               Transport -> Reliable connection
UDP	               Transport ->	Fast, connectionless
IP	               Internet ->	Packet routing

One real example: “curl https://example.com = App layer over TCP over IP”

curl https://www.google.com/
```

Hands-on Checklist (run these; add 1–2 line observations)
```
Identity: hostname -I (or ip addr show) — note your IP.

$ hostname -I
192.168.0.108 

Reachability: ping <target> — mention latency and packet loss.

$ ping google.com
PING google.com (142.251.222.142) 56(84) bytes of data.
64 bytes from pnmaaa-az-in-f14.1e100.net (142.251.222.142): icmp_seq=1 ttl=117 time=17.5 ms
64 bytes from pnmaaa-az-in-f14.1e100.net (142.251.222.142): icmp_seq=2 ttl=117 time=17.1 ms

--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 16.989/17.136/17.467/0.192 ms

Path: traceroute <target> (or tracepath) — note any long hops/timeouts.
Ports: ss -tulpn (or netstat -tulpn) — list one listening service and its port.

$ ss -tulpn


Name resolution: dig <domain> or nslookup <domain> — record the resolved IP.

$ dig google.com
142.251.222.142

HTTP check: curl -I <http/https-url> — note the HTTP status code.

$ curl https://www.google.com
301

Connections snapshot: netstat -an | head — count ESTABLISHED vs LISTEN (rough).

Pick one target service/host (e.g., google.com, your lab server, or a local service) and stick to it for ping/traceroute/curl where possible.
```

Mini Task: Port Probe & Interpret

```
Identify one listening port from ss -tulpn (e.g., SSH on 22 or a local web app).

$ ss -tulpn
Netid         State          Recv-Q  Send-Q   Local Address:Port   Peer Address:Port         Process
tcp           LISTEN         0       4096      [::]:22               [::]:*
It listens on 0.0.0.0:22, meaning the server accepts SSH connections from any network interface. 

From the same machine, test it: nc -zv localhost <port> (or curl -I http://localhost:<port>).

$ nc -zv localhost 22
Connection to localhost (127.0.0.1) 22 port [tcp/ssh] succeeded!

Write one line: is it reachable? If not, what’s the next check? (e.g., service status, firewall).

-> Yes the connection is reacheable, if it donot connect check below command
$ systemctl status ssh
```
```
Reflection (add to your markdown)

Which command gives you the fastest signal when something is broken?

$ ping google.com
Tells that you are reachable or not to the network.

What layer (OSI/TCP-IP) would you inspect next if DNS fails? If HTTP 500 shows up?

$ dig google.com
Inspect the application layer

If HTTP 500 shows up

Application crash
Backend failure
Database connection issue
Server misconfiguration

Two follow-up checks you’d run in a real incident.

$ systemctl status ssh   => check if its running
$ ss -tulpn => check if it is listening to the port
```
