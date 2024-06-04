Under construction, please ignore for now :)


Most of us have used the great RAID calculators from WintelGuy.
https://wintelguy.com/raidmttdl.pl  

But in the reliability calculator, there is one thing missing from the equation. 
The calculator assumes drive failures based on MTBF. 
While this isn't wrong, it is missing some real world variables. We will get into that later.  

For now, let us look, at the calculated probability of data loss.
I tried to fill it with variables that represent home lab users. 

Mission time 10 years. Drive capacity 16TB. 160MB/s. MTBF conservative real world. Time to replace 48h. Rebuild rate 80% for mirror, 50% for RAIDZ2. I assume we use 8 drives. 

Now let's compare the calculated risk of data loss for an 8-wide RAIDZ2 and four striped mirrors.

|     | mirror             | RAIDZ2             |
|-----|--------------------|--------------------|
| 1y  | 0.0086704138805684 | 0.0000406224288297 |
| 5y  | 0.0426067985030293 | 0.0002030956430019 |
| 10y | 0.0833982577273807 | 0.0004061500381636 |

According to this calculator, over 10 years, we have a 205x times higher chance of losing data in the mirror than in RAIDZ2. And some of you will argue, that you feel more comfortable with RAIDZ2 since any given two drives can fail. While for the mirror, two drives in the same vdev could bring down your pool. 

But the calculator has one big flaw. It calculates the chances of drives failing by MTBF. But that is not close to the real world for two reasons.

1: Not all drives are created equal. You could have a bad batch, maybe because of hardware problems, maybe because of firmware issues. Even if you do not have a bad batch but just two different drive vendors, the MTBF is not the same for every drive, but it is in the calculator.

2: The drive's annual failure rate is not static. It will probably be the bathtub curve. So in the beginning, the annual failure rate will be slightly higher, while after 5y the annual failure rate will drastically increase due to wearout. The calculator is static.

With these two points in mind, let's revisit our scenario. 
We buy new drives for our new NAS. We buy four Seagate drives and four WD drives. 
We build mirrors with one of them in each. 
For RAIDZ2 it is still one vdev. 
The Seagate batch we bought has a horrible firmware error. All four of them die after one year. This happens within days or during a resilver. 
For RAIDZ2 the pool is lost since we lost four drives. 
For the mirror everything is fine.  

In another scenario, the Seagate drives are fine, but the WDs have some issues. 8 years from now, they are extremely old. Maybe the helium in the HDD goes bad or the read heads, or the motor. Since you don't have a hot-swap, you need to shut down the system and put an additional spin-up on them. The resilver also puts an additional load on them.
For whatever reason, three of the WDs fail. 
For RAIDZ2 the pool is lost since we lost three drives. 
For the mirror everything is fine.  


I know I don't have any real hard evidence numbers to compare that with the calculator. It is to show that the calculator has its blindspots and is not able to model all variables of a real-world system. 





