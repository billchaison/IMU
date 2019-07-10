# IMU
Acquiring another identity to get through Palo Alto User-ID security policies.

If you are unable to access certain websites because they are restricted to a limited class of user then you can use this technique to gain access.  The steps described here allow an attacking Linux machine to impersonate another user's identity and trick a Palo Alto firewall into granting you access through a security policy.

**The following diagram can be used to understand the scenario**<br />
![alt text](https://github.com/billchaison/IMU/blob/master/uid.png)

**It is assumed that you have already accomplished the following**<br />
* Identified a privileged user (you do not need to know the IP address of his machine at this point)
* Have network access from your attacking Linux machine (B) is 192.168.1.5
* The remote host (C) that a user on (B) has an ssh session open to is 192.168.10.10
