# day 14 task

## Overview
**-OSI layers vs TCP/IP stack**

**osi layers: 7 layers**
1. Physical Layer (L1): Deals with the physical connection between devices, including cables, switches, and the electrical signals that transmit data.
2. Data Link Layer (L2): Responsible for node-to-node data transfer and error detection. It manages MAC addresses and controls access to the physical medium.
3. Network Layer (L3): Handles routing and forwarding of data packets across different networks. It uses IP addresses to determine the best path for data to reach its destination.
4. Transport Layer (L4): Ensures reliable data transfer between end systems. It manages flow control, error correction, and segmentation of data. Protocols like TCP and UDP operate at this layer.
5. Session Layer (L5): Manages sessions or connections between applications. It establishes, maintains, and terminates communication sessions.
6. Presentation Layer (L6): Translates data between the application layer and the network. It handles data encryption, compression, and formatting.
7. Application Layer (L7): The topmost layer where applications and services operate. It provides interfaces for user applications to access network services. Protocols like HTTP, FTP, and DNS operate at this layer.

Ip sits in the Network Layer (L3), TCP/UDP sit in the Transport Layer (L4), and HTTP/HTTPS, DNS sit in the Application Layer (L7).

**TCP/IP stack: 4 layers**
1. Link Layer: Corresponds to the OSI's Physical and Data Link layers. It manages the physical connection and data transfer between devices on the same network.
2. Internet Layer: Corresponds to the OSI's Network layer. It handles routing and forwarding of data packets across different networks using IP addresses.
3. Transport Layer: Corresponds to the OSI's Transport layer. It ensures reliable data transfer between end systems using protocols like TCP and UDP.
4. Application Layer: Corresponds to the OSI's Session, Presentation, and Application layers. It provides interfaces for user applications to access network services, with protocols like HTTP, FTP, and DNS.  

IP sits in the Internet Layer, TCP/UDP sit in the Transport Layer, and HTTP/HTTPS, DNS sit in the Application Layer.


curl https://google.com = App layer over TCP over IP means that when we run the command `curl https://google.com`, it operates at the Application Layer (L7) of the OSI model, using the HTTP protocol to communicate with the server. This communication is encapsulated over TCP (Transport Layer) to ensure reliable data transfer, and TCP itself is encapsulated over IP (Internet Layer) to route the data packets across the network to reach Google's servers and retrieve the requested web page. Later, the response from the server will also follow the same path back to the client, ensuring that the data is delivered correctly and efficiently.

## Hands-on Checklist
hostname -I: 192.168.31.58 172.17.0.1 2409:4091:0:5818:a00:27ff:fe98:6290 . After running this I found that I have multiple IP addresses assigned to my machine, including both IPv4 and IPv6 addresses. The presence of multiple IPs indicates that my machine is connected to different networks or interfaces, such as a local network and a Docker network.

ping google.com: I observed that the latency to google.com is between ms within 46-51ms with 0% packet loss, indicating a stable and responsive connection to the target host.

tracepath google.com : It showed all the hops taken to reach google.com, with some hops showing higher latency than others, which could indicate potential network congestion or routing issues at those points.

ss -tulpn: It is used to see which services are listening on which ports. I found that apache is not active so I tried to start it using `sudo systemctl start apache2` but it failed to start. After running ss -tulpn | grep 80, I found that nginx is listening on port 80, which is the default port for HTTP traffic and hence apache is not able to start because nginx is already using that port.

dig google.com: It resolved to multiple IP addresses, including both IPv4 and IPv6 addresses, which is common for large websites that use load balancing and content delivery networks (CDNs) to distribute traffic across multiple servers. It works similarly to nslookup google.com which also resolved to the same IP addresses.

curl -I https://google.com: It returned an HTTP status code HTTP/2 301 Moved Permanently, which indicates that the requested resource has been permanently moved to a new URL. This is a common response when accessing a website that has been redirected to a different URL, such as from http://google.com to https://google.com.

netstat -an | head: It showed a snapshot of the current network connections on the machine. I observed that there are several ESTABLISHED connections, which indicates active communication with other hosts, and several LISTENING ports, which indicate services that are waiting for incoming connections. 4 listen and 1 time_wait connections were observed in the output.

## Mini Task: Port Probe & Interpret

1) I identified that nginx is listening on port 80 using the command `ss -tulpn | grep 80`.
2) I tested the port using `curl -I http://localhost:80` and received an HTTP status code HTTP/1.1 200 OK, which indicates that the nginx service is reachable and responding correctly on port 80.
3) It is reachable, and the next check if not reachable would be to verify the port is free or being used already, or to check the firewall settings to ensure that port 80 is not being blocked.

## Reflection (add to your markdown)

- The command `ping` gives the fastest signal when something is broken, as it quickly indicates whether the target host is reachable and provides latency information.
- If DNS fails, I would inspect the Internet Layer (L3) to check for IP connectivity issues, such as using `ping` or `traceroute` to see if the target host is reachable. If HTTP 500 shows up, I would inspect the Application Layer (L7) to check for server-side issues, such as reviewing server logs or checking the status of the web application.
- Two follow-up checks I would run in a real incident are: 1) Check the firewall settings to ensure that the required ports are open and not blocked, and 2) Verify that the service is running correctly and listening on the expected port.
