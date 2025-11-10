## Uptime Kuma is here!
In this article, you'll learn how to monitor websites and servers using **Uptime Kuma**.

Monitoring a server using **SNMP** isn't exactly modern. Specialized monitoring software solves this problem with agents that are installed on the target system.

Smaller companies with just a few servers and limited budgets often prefer to avoid expensive and overly complex monitoring software.

:collision: To be able to react quickly to errors and failures, a few checks are sufficient, which can be accomplished with SNMP.

## Basics
Before we begin monitoring, let's start with some organizational steps:

:small_blue_diamond: Create a new **Monitor** of type **Group** for each server. Use the server name as the **Friendly Name**.<br>
:small_blue_diamond: Below this group, create the groups **SNMP** and **Websites**.<br>

:point_right: You can leave all other values ​​in the group as they are. At first, I thought these values ​​would be inherited by new objects. Unfortunately, that's not the case.

## Ping
First, we'll add a simple **ping** to each server. This is the simplest way to tell if a device is alive.

:small_blue_diamond: Create a new **Monitor** of type **Ping**.<br>
:small_blue_diamond: Enter the **Hostname**.<br>
:small_blue_diamond: I increased the **Heartbeat Interval** to *90 Seconds*. That's more than enough and reduces the load and network traffic a bit.<br>
:small_blue_diamond: To minimize false alarms, I increased the **Retries** value to **5**. I left the **Heartbeat Retry Interval** at *60 seconds*.<br>

:point_right:I prefer to use the **IP address** instead of the hostname. Why? The hostname might temporarily be unresolvable by the DNS. That would create a false alarm and raise blood pressure unnecessarily.<br>

:point_right: With this configuration, a ping alert will be triggered if no response occurs within 5 minutes.<br>
:point_right: This allows, for example, a server restart without triggering any alerts.<br>

:exclamation: We save the ping monitor directly in the server's group.

## Monitoring Websites
From a technical perspective, what kind of websites should be monitored? The database could fail, the web server could malfunction, or the software could have a problem.
I'm choosing one website per server, each with either an online store or a CMS like WordPress. Then I monitor the websites that are likely to be important.

:small_blue_diamond: Now create a monitor of type **HTTP(s) - Keyword**.<br>
:small_blue_diamond: I usually use the domain name as the **Friendly Name**, i.e., *mydomain.ch*.<br>
:small_blue_diamond: Now you need to choose a **keyword** from the website. I always choose a *word* or *phrase* from the very bottom of the website.<br>
:small_blue_diamond: In the **URL** field, you enter the full address of the website: *https://www.mydomain.ch*.<br>
:small_blue_diamond: In the **Heartbeat Interval** field, I set the value to *300 seconds*.<br>
:small_blue_diamond: I leave the **Retries** value at *1* and the **Heartbeat Retry** value at *60*.<br>

:exclamation: We store this monitor under **Monitor Group** *\<ServerName\>/Websites*<br>

:point_right: I don't see the point in checking the website every minute. That only increases the chance of false alarms.<br>
:point_right: With this configuration, an alarm is triggered after **10 minutes**. After that, checks are performed every minute.<br>
:point_right: This allows for brief website maintenance without causing panic.<br>

:exclamation: Some websites cannot be analyzed using a keyword. In those cases, I use the simple monitor type **HTTP(s)**.<br>

## Monitoring with SNMP 
Now it gets interesting and also a bit complicated. Generally speaking, SNMP is a pain in the ass.

I assume you've already configured SNMP on the servers. **UDP port 161** must be open on the firewall. I'm limiting this to the IP address of the server running Uptime Kuma.

### Monitoring Memory
We are interested in the available free RAM. We want to know what percentage is still available.

First, let's find out how much RAM the server has available: Enter `free` on the command line. The result will be given in *KB*.

To enable Uptime Kuma to read the available memory using SNMP, we need to use the appropriate *Object Identifier*: **1.3.6.1.4.1.2021.4.6.0**.<br>
We can try this manually right on the command line:

```
snmpget -v 2c -c <SNMPCommunity> localhost 1.3.6.1.4.1.2021.4.6.0
```

For *\<SNMPCommunity\>*, you enter the value that is stored as *community* in the file `/etc/snmp/snmp.conf`. I'm sure you're not using *public*, right?<br>
As a result, you will get something like this:

:small_blue_diamond: Now let's create a new monitor of type **SNMP*.<br>
:small_blue_diamond: You could use *Free Memory* as the **Friendly Name**.<br>
:small_blue_diamond: Enter your server's **IP address** as the **Hostname**.<br>
:small_blue_diamond: We will leave the *Port* and the *SNMP version* at their default values.<br>
:small_blue_diamond: Now enter the name of your *SNMP community* in the **Community String** field.<br>
:small_blue_diamond: Enter *1.3.6.1.4.1.2021.4.6.0* as the **OID (Object Identifier)**.<br>

```
UCD-SNMP-MIB::memAvailReal.0 = INTEGER: 1005112 kB
```
This value should be approximately the same as the value of *free* from the previous command.

