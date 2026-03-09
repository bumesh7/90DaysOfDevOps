Challenge Tasks
Task 1: DNS – How Names Become IPs

    Explain in 3–4 lines: what happens when you type google.com in a browser?

    1) Whey you type google.com, browser asks OS for IP address.
    2) Then OS checks cache, the quries DNS resolver and which may ask root and other DNS servers.
    3) DNS server returns the IP address.
    4) Browser connect to IP via HTTP/HTTPS.
    
    What are these record types? Write one line each:
        A, AAAA, CNAME, MX, NS

        A => Maps domain name to an IPv4 address.
        AAAA => Maps domain name to an IPV6 address.
        CNAME => Alies one domain name to another domain name.
        MX => Mail service responsible for receiving mail from domain.
        NS => Translates domain names into IP addresses that computers use for communication. Ex = example.com
        
    Run: dig google.com — identify the A record and TTL from the output
    
    google.com.		300	IN	A	142.251.223.174

    Record => 142.251.223.174
    Time To Live = 300


Task 2: IP Addressing

    What is an IPv4 address? How is it structured? (e.g., 192.168.1.10)

    -> It is a 32 bit numeric address used to store identify device on a network.
    -> It has 4 octets seperated by dots. Each octet 8 bits.
    Ex => 192.68.10.24 
    max => 2^32 = 4,294,967,296
    
    Difference between public and private IPs — give one example of each

    -> Private IP are used with in the network ex => 8.8.8.8
    -> Public IP are used outside the network and it is provided by ISP (Internet Service Provider). Ex => 192.68.12.27
    
    What are the private IP ranges?
        10.x.x.x, 172.16.x.x – 172.31.x.x, 192.168.x.x

        10.0.0.0 - 10.255.255.255
        172.16.0.0 - 172.31.255.255
        192.168.0.0 - 192.168.255.255
        
    Run: ip addr show — identify which of your IPs are private

    10.255.255.254  
    172.17.191.255

Task 3: CIDR & Subnetting

    CIDR => Classless INter Domain Routing
    It is the method of alloctaing IP address and routing that allows efficient use of ip addess.
    
    What does /24 mean in 192.168.1.0/24?

    -> out of 32 bits, 24 bits are used for network address and 8 bits for host address.
    
    How many usable hosts in a /24? A /16? A /28?
    /24 = 32-24 = 2^8 = 256 - 2 = 254
    /16 = 32-16 = 2^16 = 65,536 - 2 = 65,534
    /28 = 32-28 = 2^4 = 16 - 2 = 14

    1 - network address
    1 - briadcast address
    
    Explain in your own words: why do we subnet?

    -> Subnet is the process of dividing large number of IP network into smaller logical network.
    -> Helps in security control, improve performance, correct usage of IP address.

    CIDR 	Subnet Mask 	Total IPs 	Usable Hosts
    /24   255.255.255.0  256        254
    /16   255.255.0.0    65,536     65,534
    /28   255.255.255.240 16        14

     240 = 32-28 = remaining 4 octets = > 128+64+32+16 = 240 

     1   2  3  4  5   6  7  8
     1   1  1  1  1   1  1  1  = > 8 octets
     128 64 32 16 8   4  2  1  => 2^   
    Quick exercise — fill in:

CIDR 	Subnet Mask 	Total IPs 	Usable Hosts
/24 	? 	? 	?
/16 	? 	? 	?
/28 	? 	? 	?

Task 4: Ports – The Doors to Services

    What is a port? Why do we need them?
    
    -> Port is a logical address of 16 bit unsigned integer that is allocated to every application on the computer 
       that uses internet to send or receive data.
       
    Document these common ports:
```
Port 	Service
22 	  SSH
80 	  HTTP
443 	HTTPS
53 	  DNS
3306 	MYSQL
6379 	Redis
27017 MongoDB
5432  Postgress
```
    Run ss -tulpn — match at least 2 listening ports to their services

    22 and 3306

Task 5: Putting It Together

Answer in 2–3 lines each:

    You run curl http://myapp.com:8080 — what networking concepts from today are involved?

    -> DNS resolves myapp to an IP address and TCP(Trasmission Control Protocol) establishes conection to port 8080.
    -> HTTP protocol is used to request the data.
    -> Routing may control packet delivery across network.
    
    Your app can't reach a database at 10.0.1.50:3306 — what would you check first?

    -> Check MYSQL is listening to port 3306
    -> check network connection ping the server.

