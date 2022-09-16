# EldenRing-Notes
Observed network behavior in EldenRing CoOp

For the past several weeks I've been running packet captures while initiating cooperative play in Eldenring. 

Summoning a duelist. (Red sign on the ground.)
Being summoned for a duel.
Being summoned as a hunter.
Invading other players.
Summoning other players for help. (Golden sign on the ground.)
Being invaded by other players.



What I've learned is that all of this traffic looks almost identical from a connection setup perspective. These two firewall rules I've applied log the remote IPs. I'm using PFSense.
1662393862 = Applied on LAN interface
IPv4 UDP traffic from any host and udp port 8571 to any host and udp port 8571
grep -e 1662393862 /var/log/filter.log | cut -d "," -f 20



1662557376 = Applied on WAN interface
IPv4 UDP traffic from any host and any port, to ps5 internal IP and udp port 8571
grep -e 1662557376 /var/log/filter.log | cut -d "," -f 19

The numbers are arbitrary rule numbers on my firewall, but they are logged and this is how I can extract those IP addresses.

What generally happens is that the eldenring servers will broker the connection, therefore the server will be speaking out first. I noticed this traffic happens on UDP 3478 on the server side, client side retains 8571udp. I'm going to assume it tells me the other player's IP address and I attempt to reach out directly.  After the first few packets with the Eldenring server my PS5 starts to talk with a public IP on 8571, and depending on the circumstances, sometimes it simultaneously tries to talk to an RFC1918 address. I can only surmise this is the private IP address of the remote player's PS4/5.

Some other edge cases I've seen is different port assignments with carrier grade NAT users.  I had one invasion on a group of three players in Japan, my packet capture only showed 1 ip and port, the port was a higher port, not 8571, and ALL three of these users were working over the same remote IP and port.

Lesson Learned:
When you are being invaded, or invading someone, you may not always be able to tell how many people you are walking into at the time on the other side, but that is the exception and not the rule. Most of the time you have an individual connection to each.

What can you do with this data?
You can see what kind of latency they have, or at least have a good idea.  Not all end user IPs will respond to ping or traceroute, but if you run a traceroute to the IP and the last few hops before it starts returning timeouts are 200+ms, you know you're dealing with either a long distance player or someone with a bad connection IMO.
You could also use it to tell if you're facing the same person over and over, they might change accounts. It's a good indicator. Technically I think you could face a different person in this situation using the same exact IP and port from earlier, but that would be highly unlikely based off of what I've seen and how I understand this.

How can other people use this data?
Well, if you get a particularly unscrupulous user on the other side who is doing their best to invade you during a live stream, they may catch your IP address and initiate a DDoS against your connection.  Unfortunately there aren't a lot of great ways to mitigate this, you could try a VPN, but I have not tried this, and I'm not sure if your PS4/5 will end up leaking your IP in one way or another like it leaks your internal IP for connection purposes.  The internal IP is probably for the off chance you're cooping with someone else on the same LAN segment.
