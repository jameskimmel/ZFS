There seems to be a lot of misunderstanding when it comes to PLP.  
PLP is not about securtiy, but about performance.  
And no, PLP has nothing to do with your UPS. 

To understand why, we need to understand how PLP and none PLP SSDs work.  

A normal SSD without PLP will perform pretty poor when it comes to sync writes because it can not use  
any cache. It really has to write down the sync write to NAND. Even if the SSD has some kind of DRAM cache,  
it can't make use of it for sync writes. DRAM is volatile, if the system halts for whatever reason, the data in DRAM is lost. A UPS will only protect you form a power loss, not from a system halt. That is why your UPS has nothing to do with PLP.  

Now lets look at a PLP SSD. This SSD will report back to the system that a sync write is done, when in reality the write is still in DRAM and not NAND. Why does this drive basically lie about the write?  
It is because the drive has PLP. PLP is a combination of firmware and hardware but basically there are capacitors that help the drive finish the write even when the power is gone. 

So no, PLP is NOT added security. PLP protects the "lying about cache" part, a "problem" that drive created for itself to gain performance. 
Saying that PLP is "added security" is like putting a turbo on your engine and control it via software and then claim you added security to your engine.  
No, you protected your engine from a source of failure that wasn't there to begin with.  

With that knowlege, it is easy to understand why none PLP perform so poorly compared to PLP drives. 
Take a look at the Samsung 850 EVO and compare that to the Intel S3510.

![1692011635190](https://github.com/user-attachments/assets/9a3b12b5-1a82-46d5-8454-85effe30036d)

339 IOPS of a Samsung 850 EVO vs 12,518 IOPS of the Intel S3510. 
On paper the 850 EVO is advertised with 98,000 IOPS while the S3510 is only advertised for 20,000 IOPS.
The Evo can only handle the load for a few seconds (and only if the drive has enough free space, so it can use pseudo SLC cache) until the performance will heavily drop. 

That is why for SLOG enterprise PLP drives are recommended. Not because of security.


A word of caution: 
In the past, there were none PLP SSDs that lied about sync. A twitter found out by running some test. Unfortunately the twitter user deleted his account, so I can't share it with out. Some Phison E18 controllers in combination with bad firmware from ADATA and Patriot lied about sync. That is why I don't blindly trust cheap consumer drives.
