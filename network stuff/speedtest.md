# Why speedtest.net is almost meaningless

## How does speedtest.net work?
One of the most common tools measure network performance is iPerf3. 
The Speedtest webpage or App is basically an iPerf3 speedtest from your PC (or tablet or smartphone or whatever), to your modem/router, to some iPerf server in your ISPs core network. 
This looks something like this:
![367361535-83f9bbe9-20e4-4717-a250-32ea7af2002f](https://github.com/user-attachments/assets/906c31bb-b52f-410b-8322-e606dae1c914)

So the only thing you are measuring is the speed from you device to you router, how fast you router can handle traffic, and how fast your connection to some server in your ISPs network is. 

**The most immportant thing cant be measured by speedtest; peering!**

## What speedtest.net can't measure
Speedtest can't measure how good over good your connection to Netflix is.    
Speedtest can't measure how good ping from your PC to the Call of Duty multiplayer server is.  
Speedtest can't measure how fast you can download a game on steam or the new iOS Update from Apple.  
Speedtest can't measure overbooking from your ISP. You can read more about overbooking here:  
https://blog.init7.net/en/overbooking-how-providers-divide-up-the-bandwidth  
Speedtest also can't measure how good the peering to someone else is. If you make a video call with someone else, a lot of software will use peer to peer (P2P) for that. So you have a direct connection from your PC to someone elses PC.

This could look something like this:
![367704799-0d429301-54cb-4d77-add3-03228753dd53](https://github.com/user-attachments/assets/bdbee955-4d3d-407a-a407-da454fe5de0d)



Compare that with the picture above and you will see how many dotted "connection lines" are not measured in a simple speedtest.net benchmark.  

**That is why you can't differeantiate an ISP with good peering from an ISP with bad peering just by using speedtest.net results.**  

I have seen 1GBit lines not being able the stream a 15MBit video from Netflix,  10Gbit lines being slower in downloading Cyberpunk 2077 at launch than a 50Mbit line, or employess being unable to establish a stable VPN connection to their office. 


Side note:  
ISPs can install cache servers in their network, have direct peering with other ISPs and use public peering exchange points to offer you a better "Internet".  
There are also content delivery networks (CDN) that try to alleviate the performance bottlenecks of the Internet.  
Unfortunatly, the internet is a lot less decentralized than some people think it is. 







