Wi-Fi is a wonky construct, full of bugs and badly written firmware. You wonder how bad it is? Well, [even companies like Google struggle with Wi-Fi](https://www.reuters.com/technology/googles-newest-office-has-ai-designers-toiling-wi-fi-desert-2024-03-07/).
There are millions of combinations between devices and access points and what OS they are currently running. Because of this, the best way to troubleshoot Wi-Fi and eliminate potential pitfalls is to reduce complexity.

**These are my recommended settings:**

Split your Network into two. One network is for old clients or IoT stuff like your Ring doorbell or your fridge. The other network is for your "real" devices that need performance,   
like phones, laptops, smart TVs, and so on.  
So assuming we name your Wi-Fi "SunnyGarden" we will create two SSIDs with these settings:  

SSID: SunnyGardenIoT  
- 2,4GHz only  
- WPA2  
- 20MHz channel width  
- SSID: SunnyGarden  

SSID: SunnyGarden  
- 5GHz only  
- WPA3 only  
- If WPA3 only is not possible, WPA2 only.   
- No WPA2/3 mixed mode!
  
channel settings for SunnyGarden if you expirience problems: 
- channel width will be selected based on how busy your neighborhood is. But in general, if you don't need a lot of bandwidth, like sending huge files from your Laptop to your NAS, choose 20MHz.
- Channels should be automatically selected, based on your neighbors Wifi. If the channels 38 or 46 are not busy, you can select one of these channels with 20MHz. These are the only two channels that don't need DFS to detect radar. I saw some buggy firmeware implementations for DFS from AVM. By not using DFS, you can rule out these errors. 

Not every AP will allow you to configure all of these settings.
These settings are not written in stone and are mostly valid for Wi-Fi 6. They could change over time. 
For example, Wi-Fi 7 should solve some problems like bandsteering, since it can be connected to 2,4 and 5GHz at the same time. 

**FAQ**  

*In theory, feature X should not be a problem. X is also backward compatible!*

In theory, a lot of things are true. But remember, Wi-Fi is a mess. In theory, HomePod Minis should be able to connect to both WPA2 and WPA3. 
But in practice,  HomePods will sometimes not connect to a WPA2/3 mixed mode Wi-Fi from vendors like AVM FritzBOX.  

*Why not use WPA2/3 mixed mode?*

See above

*Why not use 2,4GHz and 5GHz combined?*

Band steering is an added layer of complexity that sometimes does not work well. 
My rather controversial opinion is that if I can't get 5GHz, I rather have no connection at all. 2,4GHz has bad latency and so many interfering Wi-Fis, that it is mostly useless besides IoT devices.
It is also not great for VoIP, because it takes some time to switch between bands. If I leave my house on a call, I rather want my phone to switch to cellular directly without connecting to 2,4 in between. Wi-Fi 7 should address that issue, by allowing clients to simultaneously connect to both bands. 

*But the 5GHz-only network does not reach my garden!*

The higher the bandwidth, the smaller the range and object penetration. That is physics. There are two solutions to this problem. You could either install another cable(!) connected extender to expand your range, or you could manually switch to your SunnyGardenIoT and set your device to NOT auto-connect to that Wi-Fi. Otherwise, your device may connect to the slow Wi-Fi inside your home.

*Why does an extender have to be connected by cable?*

There will be a cable either way because the Mesh or AP needs power from somewhere. In my opinion, it is in many cases easier to connect an AP or mesh node by PoE (LAN cable that also transmits power from a PoE-capable switch) than by a wall outlet. 
What most people have a hard time understanding, is that an AP or a mesh node is just a repeater. It can only repeat what it received previously. Bad input = bad output. 
Remember that I think that 2,4 is mostly useless? Well, some manufacturers like Unifi or AVM think so too. That is why they won't offer the repeater getting 2,4GHz as input, only 5GHz. But that gets us to another problem, 5GHz has a pretty limited range. So you have to place it pretty close to your router or use multiple repeaters.
If possible, always use a cable connection.  

*Where should I place my repeater?*

Let us look at the typical errors I see when people install a mesh node. For simplicity, we leave out 2,4GHz.

![Unbenanntes Diagramm drawio](https://github.com/jameskimmel/Wi-Fi/assets/17176225/34f24712-8576-4b58-b5b5-a9cdb634c768)

What we see is the router on the left and how good the signal quality is in other rooms. 
Now if we install a repeater in the room3, the signal the repeater gets would be poor. But since the iPhone in the garden is one wall away from that repeater, it would show a good signal. 
Which is true, it has good signal, but to a repeater that receives poor signal. So in the end how have a poor expirience but your iPhone will show everything is great.  
So if we really can't use a cable connection for the repeater we should place it in room 2 or even better in room 1 and another one in room 2 and 3. That way you get good signal everywhere and ok signal in the garden.  Or you could save a lot of money and energy by just installing a wired repeater in room 3 and get good signal everywhere including the garden. 

