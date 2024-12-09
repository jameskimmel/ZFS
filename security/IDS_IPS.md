Copied from here  https://forum.opnsense.org/index.php?topic=44259.msg221025#msg221025

Mostly agree, altough I still think it can be usefull for detection. 

Here is a copy of the message:

1. Trying to exaustively "enumerate badness" is bound to fail.

The rulesets of current IDS/IPS systems are ridiculously large and generate a ton of false positives. They place a high administrative burden on the operator in the form of tailoring the rules.

As system that is always "flashing red" but operators "know what can be ignored" is useless. Any monitoring system must be "green" in the normal operational state and any "red flag" must be an event that demands action - and can be acted upon in the first place! That means there is a documented way to turn the system back to the "green" state.

Anything else is security theatre and leads to more confusion than it helps.


2. It's never a one-stop solution.

People tend to think that all they need to do is "enable intrusion prevention" and then someone else will prevent intrusions for them. This is not the case for the reasons listed under 1.

From the same reasoning probably we see people trying to activate Suricata and Zenarmor on the same device (because two IPS are better than one, right?) and face technical problems like in this thread - here's a nice comment by Ad about architectural issues:

https://forum.opnsense.org/index.php?topic=9741.msg64178#msg64178

The way IDS/IPS works it is highly unlikely it will ever be able to inspect a PPPoE data stream for example. Also both products have a different focus. So why would you run both (at all) and if it does make sense for you, why both on WAN (doesn't work!)?

So if you still decide to run them at all, separate your publicly reachable servers from your client systems that are merely "facetubing". Use Suricata for the servers (on the server/DMZ interface) and Zenarmor for the clients (on the LAN interface).


3. Traffic is encrypted nowadays.

Most interesting Traffic is encrypted in some form of TLS/SSL. And generations of cryptographers and protocol specialists have been working hard to make TLS secure against man-in-the-middle attacks.

A TLS encrypted session is an authenticated secure private channel between e.g. your browser and the web site of your bank.

If you insist on messing with that so the holy IDS can perform its "magic", you will actually

- weaken the security of these connections
- force all devices to trust your private CA leading to a very interesting attack vector against your entire infrastructure
- make all applications that use certificate pinning fail, so you need to curate extensive white lists

TLS is designed not to be inspected. What's so hard to understand about that? Just don't.


4. But but but ... I must do $something to improve "security".

So:

- separate systems of different trust.
- implement firewall policies with least privielege principle - give the system access to only what they absolutely need.
- implement a log file and/or reputation based system like fail2ban, Crowdsec, ...
- implement AdGuard Home or some other DNS based block & report mechanism.
- do monitor what is happening around your network, use an NMS like Observium, NtopNG, some Elastic based solution like pfELK - "The number of times an uninteresting thing occurs is an interesting thing." (Marcus Ranum, IIRC, on firewall-wizards).
- but don't worry about trash arriving at your front door (WAN) that is blocked by the firewall, anyway - blocked is blocked, an IPS does not block better or more effectively or anything than the default "deny all" rule.


Kind regards,
Patrick


EDIT: here's a paper worth reading that discusses the pros and cons of TLS interception:
