# squid_squidGuard_Mikrotik_pi
Running squid and squidGuard on a pi transparently with a mikrotik router
Always wanted to set up a squid and squidguard transparent proxy on your Mikrotik router, and a raspberry pi? Then look no further!


To get squid up and running on your PI, check out this guy on YouTube (3 parts). He explains it like a beast. Nobody can beat him:
https://www.youtube.com/watch?v=j7rrHHR0qYw

To get a proper squidguard list of IPs and domains, look around the internet. I am not going to share mine as it has too many false positives.

# Now for the other half:

### Mikrotik Rules:
/ip firewall mangle chain=prerouting action=mark-routing new-routing-mark=to_proxy passthrough=yes protocol=tcp src-address=YOUR_IP_HERE dst-port=80
--Replace YOUR_IP_HERE with an actual IP or subnet
--Add port 443, 8080, etc


--Now lets route this marked packet to our raspberry pi running Squid:
/ip route add disabled=no distance=1 dst-address=0.0.0.0/0 gateway=YOUR_SQUID_IP routing-mark=to_proxy scope=30 target-scope=10
--Replace YOUR_SQUID_IP with your squid server's actual IP address


### Linux Rules (on your IP):
On your raspberry pi that is running squid, add this IP tables rule:
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to YOUR_IP:YOUR_PORT
--Replace YOUR_IP and YOUR_PORT with the raspberry pi's IP and the port your squid is running on (should be 3128)


Now on the device with YOUR_IP_HERE, open a website (if you specified port 80, open a non-SSL page). 
But oh no! You're getting a 400 bad request! WHAT EVER SHOULD I DO NOW??!! Easy! Make sure in your squid.conf file you have this:
http_port 3128 transparent

Without the word "transparent" squid will throw away the domain portion and just work on everything after the domain and that's why it freaks out.
