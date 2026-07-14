# Day 15 task

### Task 1: DNS – How Names Become IPs

1. When we type `google.com` in a browser the dns resolver on our computer sends a query to a DNS server to find the corresponding IP address for `google.com`. The DNS server responds with the IP address and then checks if there are any cached records. If not, it may query other DNS servers until it finds the correct IP address. Once the IP address is obtained, the TCP connection is established to the server hosting `google.com` and the browser sends an HTTP request to load the website. Then the server responds with the website data, which is rendered in the browser.

2. - `A`: Maps a domain name to an IPv4 address.
    - `AAAA`: Maps a domain name to an IPv6 address.
    - `CNAME`: Canonical Name record, used to map one domain name to another (alias) instead of an IP address.
    - `MX`: Mail Exchange record, specifies mail servers for receiving email for the domain.
    - `NS`: Name Server record, indicates which DNS servers are authoritative for the domain.

3.After running `dig google.com`, I found the A record is 142.251.43.46 and the TTL (Time To Live) is 21 seconds which means that the DNS resolver can cache this record for 21 seconds before it needs to query again for the same domain.

### Task 2: IP Addressing

1. An IPv4 address is a 32-bit numerical label assigned to each device connected to a computer network that uses the Internet Protocol for communication. It is structured as four octets (8 bits each) separated by dots.
2. - Public IP: An IP address that is accessible over the internet. Example: `www.google.com` has a public IP address.
   - Private IP: An IP address used within a private network, not accessible from the internet. Example: `172.23.100.32` is a private IP address.
3. The private IP ranges are:
   - `10.0.0.0` to `10.255.255.255`
   - `172.16.0.0` to `172.31.255.255`
   - `192.168.0.0` to `192.168.255.255`

4. After running `ip addr show`, I identified that my private IP addresses are `10.255.255.24` and `172.23.100.32`

### Task 3: CIDR & Subnetting

1. The `/24` in `192.168.1.0/24` indicates that we can create 256 IP addresses in this subnet, with 24 bits used for the network portion and 8 bits for the host portion.
2. There are 254 usable hosts in a `/24` (256 total IPs minus 2 for network and broadcast addresses), 65,534 usable hosts in a `/16`, and 14 usable hosts in a `/28`.
3. We subnet to efficiently manage and allocate IP addresses within a network. Subnetting allows us to divide a larger network into smaller, more manageable sub-networks, which can improve performance and security by isolating different segments of the network.
   
| CIDR | Subnet Mask | Total IPs | Usable Hosts |
|------|-------------  |-----------|--------------|
| /24  | 255.255.255.0 | 256       | 254          |
| /16  | 255.255.0.0   | 65,536    | 65,534       |
| /28  | 255.255.255.240 | 16      | 14           |

### Task 4: Ports – The Doors to Services

1. A port is a virtual point where network connections start and end. They are used to identify specific processes or services on a device, allowing multiple services to run on the same IP address without conflict. Ports help direct traffic to the correct application or service.
2. Common port numbers include :80 (HTTP), 443 (HTTPS), 22 (SSH), 53 (DNS), 3306 (MySQL), 6379 (Redis), and 27017 (MongoDB).
3. Port	Service	Evidence from Output
80	HTTP (Web Server)	tcp LISTEN 0 511 0.0.0.0:80
53	DNS (Domain Name System)	tcp LISTEN 127.0.0.53:53 and udp 127.0.0.53:53

### Task 5: Putting It Together
1. When I run `curl http://myapp.com:8080`, several networking concepts are involved. First, the DNS resolution process occurs to translate `myapp.com` into its corresponding IP address. Then, a TCP connection is established to the server hosting `myapp.com` on port 8080. The HTTP protocol is used to send a request to the server then the load balancer or web server processes the request and sends back a response, which is then displayed in the terminal. This involves concepts of DNS, IP addressing, ports, and the HTTP protocol.

2. I will first ping the server to check if it is reachable. Then I will use `curl` to send an HTTP request to the server on port 8080 and check the response. If there are issues, I will check the firewall settings to ensure that port is open and not blocked and is not used by any other service. Additionally, I will verify the security group/nsg settings if the server is hosted on a cloud platform, and check the server logs for any errors related to the database connection. Finally, I will ensure that the database service is running and listening on port 3306.

Today I learned:
1. The DNS system is crucial for translating domain names into IP addresses that computers use to identify each other on the network and there are different types of DNS records that serve various purposes.
2. Ports are essential for directing network traffic to the correct services on a device, and there are common ports associated with specific services that can be used to identify what is running on a server.
3. CIDR notation is a method for allocating IP addresses and subnetting, which helps in efficient network management and allows for a varying number of hosts within a subnet.