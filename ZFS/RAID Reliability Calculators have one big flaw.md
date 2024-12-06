Most of us have used one of these great RAID calculators.
https://wintelguy.com/raidmttdl.pl  
https://jro.io/capacity/

But there is one thing missing from the equation. 
These calculators assume drive failures based on MTBF. 
While this isn't wrong, it is missing some real world variables. We will get into that later.  

For now, let us look, at the calculated probability of data loss, according to wintelguy.  
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

I know I don't have any real hard evidence numbers to compare that with the calculator. It is to show that the calculator has its blindspots and is not able to model all variables of a real-world system. And that you really should get drives from different batches :)

If you wanna dive deeper into these numbers and like to have some fun with assumptions that I just pull out of thin air, 
[here is what I wrote in the TrueNAS forums]([https://duckduckgo.com](https://forums.truenas.com/t/raid-reliability-calculators-blindspot/5781/12))

Hi jro, thanks for taking the time.

I used your awesome calc before! Thanks for that!

So I even skipped the attached PDF and that also seems to clearly point to the bathtub curve.
They also have not found a strong correlation between temps/usage and AFR. My guess (not said in this study) is that age plays a more important factor.
Batches have different AFR rates, but in the batches themselves, drives tend to fail close to each other. This further gives me the feeling that this is something that gets overlooked.

I played with your calculator. Maybe I am too tired from work, but here is an idea.
AFR calculates over a whole year (duh!).
So even if we assume that all drives will fail in a certain year and set a failure rate of 100%, it is still spread out over a whole year. So for 8 drives that would be on average 45 days apart.
That seems pretty generous for a batch that is breaking down.
So how about we set resilver time from 48h to 960h?
That is a 20 times increase, does that give us the same number as 100% of the drives failing within 18.25 days (365 / 20)? Will have to check if the math works out on that tomorrow :joy:

Because in that case, mirror is safer than RAIDZ2 according to your calculator. And we haven’t even looked into the batch problem.

The batch problem:
Assuming this:
Seagate have an AFR of 5% at year 5 and 100% rate at year 6.
WDs have an AFR of 100% at year 5.
We don’t know these numbers beforehand, because our time machine is broken :smile:

Assuming we only got Seagates:
We have a 66% chance of zero failures, but it could also be one or all of them. Assuming we are “lucky” and one drive fails in this year. Why this is lucky, we will see later.
Anyway, we replace that drive in the year 5 and thous have a fresh disk in the pool. If we assume a 0% AFR (just for simplicity) for the first year of that drive, the AFR is down to 87.5% for year 6.

mirror 3.628%
RAIDZ2 3.414%
Here, RAIDZ2 is more secure.

Assuming we only got WDs:
We have a 100% AFR for the year 5.

mirror 4.718%
RAIDZ2 4.826%
Here, mirror is more secure. But for both pool configs, the above example is more secure, thanks Seagate spreading the risk out better by failing one drive a year before and get AFR a little bit down by doing so. That is why we were “lucky” that one Seagate drive failed the year before.

Assuming we got two mixed batches and we are looking at year 5:
Our combined or mixed AFR would be 52.5%? Again have to check the math on that tomorrow :slight_smile:

mirror 1.318%
RAIDZ2 0.857%
Two things I notice here.
A: Since the failing of drives is more spread out over the years, chances of pool failure go down significantly. That alone should be a huge reason factor for your resiliency calculations. But we almost never talk about that.

B: It might seem like RAIDZ is better at first glance, since the percentage number is lower, but it is not really true. We used 52.5%, but that is just the average failure rate and not what really happens. We have four drives with AFR of 100% and four drives with 5%.
Lets assume the worst case scenario. The four WDs fail on the first day of the year, which would be very unlucky. But it could happen, imaging a firmware bug or some mechanical problems. We would loose all of our data in a RAIDZ2. Not so for mirror. Data is still there, but the four drives left are basically are a 4 drive stripe with a 5% AFR until the resilver is done.

Assuming resilver takes 48h, we have eight days with 4 disks at risk, six with 3 disks at risks, four days with 2 disks at risk and two days with one disk at risk.

19% risk for 8 days = 19/365 * 8 = 0.416% AFR
15% risk for 6 days = 15/365 * 6 = 0.246% AFR
10% risk for 4 days = 10/365 * 4 = 0.109% AFR
5% risk for 2 day = 5/365 * 2 = 0.027% AFR
Total = 0.798%

Even though this is the absolute worst case for the mirror scenario, is still lower than the 0.857% of RAIDZ2.

If we compare a “single batch” system with a “two batch” system, the advantage of the two batch is, that you don’t have these extremes when it comes to AFR. That is thanks to drive failures being more spread more out.

