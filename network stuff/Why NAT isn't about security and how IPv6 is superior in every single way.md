IPv4 people have a hard time wrapping their heads around the fact that NAT is not about security but a wonky workaround for an IPv4 weakness.    
There is such much confusion around IPv6, mainly because people try to reimplemented IPv4 stuff onto IPv6.  
Don't try to apply your IPv4 knowledge and wisdom onto IPv6. It won't work.  
In my opinion, the easiest way to understand the topic is by using a simple example.   

**Let’s assume I want to remotely access the webpage of my NAS.**

Probably not a great idea, since my NAS is probably very insecure, full of bugs, and has no 2FA nor any brute force protection. 
And without certs, there also could be a man in the middle attack without me noticing.  
But still, I wanna access my NAS webpage from remote.  

For IPv4 I have a problem. Since my ISP only gave me one single IPv4, I use that IPv4 for my router.  
I don’t have another IPv4 for my NAS (unless a pay huge amount for business lines with multiple static IPv4). So I need the wonky workaround NAT.  
My router makes DHCPv4 and hands out my NAS the static IP 192.168.1.2.  
I say my router that all the traffic from Port, let’s say 8000, should be redirected to 192.168.1.2.  
That of course also creates the corresponding firewall rule to allow all WAN (on port 8000) to 192.168.1.2 (also on Port 8000).  
This firewall rule is the potentially dangerous part.  
Just to be clear, in theory that should not be a problem!  
It isn’t the firewalls job to protect my NAS. Sure it is nice that I don’t allow incoming traffic to my NAS if I never use that from remote,   
but the devices **themselves** should be secure.  
But of course that is not how it works in reality and because of that firewalls offer added security.  
So with that firewall rule to our NAS, we weakened our security and the NAS is now prone to brute force attacks and exploits.  

Since my IPv4 address probably is not static unless I pay my ISP, I need to setup DynDNS.  
No problem, I get a DynDNS provider and create an A record.  
Now I have access my camera over NAS.mydomain.com:8000.  

Great, there are just a few problems with that. There is now a public DNS record of my NAS. 
Portscanners can also detect that I have opended port 8000 and will try to brute force it.  

Now let’s compare that with IPv6.  
My ISP gives me a static /48 prefix. Lets say 2000:1111:1111:1111:/48  
My NAS get’s its own IPv6.  
Maybe my NAS has IPv6 privacy extension. If that is the case, we get three IPv6 adresses.  
We use the second IPv6, because that one is (unlike the first one) static and publicly routable (unlike the third one).     
We could also disable privacy extension instead, but I would not recommend it.  
Let’s say the NAS IPv6 is: 2000:1111:1111:1111:7e89:dbf4:972a:b685.
Now I create a firewall rule that allows all traffic from WAN to 2000:1111:1111:1111:7e89:dbf4:972a:b685.
Notice how we did not have to bother with DHCP! There is no DHCP server but my NAS will still get always the same IP.  
To access my NAS from remote, I simply use a bookmark in my browser or type [2000:1111:1111:1111:7e89:dbf4:972a:b685].  
The brackets tell the browser that this is an IPv6.  

Now, comparing that to IPv4, I have no public DNS record!  
There will be no port scans, since that is not feasible for IPv6.  
Even if an attacker knows my static /48 prefix, it has to scan 65536 /64 subnets, each with 18446744073709551616 IPs. Good luck with that.  

So as you can see, there is already a security benefit there.  
That lead some ISPs to even change the default firewall rules.  
Unlike with IPv4, where you by default block ALL incoming connections, these ISPs think it so unlikely that someone does IPv6 attacks, 
the changed the default from “block all incoming IPv6” to “allow all incoming IPv6,  
but just to be safe, don’t allow some very niche pro ports that a normal users probably does not want to be public like SSH or RDP”.  

Don't try to apply your IPv4 knowledge and wisdom onto IPv6. It won't work.  
If you take one of your problems step by step with a fresh mind, you will see that it isn’t that complicated and IPv6 has way less wonky workarounds.  
Most of the time, you don't even need to configure anything in IPv6 and it will work just out of the box.  
Here are some services that are most of the time no longer needed in IPv6:  
- A DHCP server: In IPv6 devices can self assign an IP so they don't need DHCP  
- A static DHCP leases or manually setting a static IPv4 on the device: In IPv6, devices already have two static IPv6s  
- NAT: That was just a wonky workaround because you only got one IPv4  
- The security nightmare UPNP: No longer needed, since we don't do NAT  


Not all software is ready for a changing prefix. pfSense is not ready, OPNsense is.  
All none shite ISPs will give at least at least a /56 static prefix.  
