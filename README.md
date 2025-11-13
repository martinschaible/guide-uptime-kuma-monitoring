## Uptime Kuma is here!
In this article, you'll learn how to monitor websites and servers using **Uptime Kuma**.

Monitoring a server using **SNMP** isn't exactly modern. Specialized monitoring software solves this problem with agents that are installed on the target system.

Smaller companies with just a few servers and limited budgets often prefer to avoid expensive and overly complex monitoring software.

:collision: To be able to react quickly to errors and failures, a few checks are sufficient, which can be accomplished with SNMP.

## Basics
Before we begin monitoring, let's start with some organizational steps. To keep track of everything, it makes sense to store the different sensors in appropriate *folders*. We use a **Group Monitor* for each one:

:small_blue_diamond: Create a new **Monitor** of type **Group** for each server. Use the server name as the **Friendly Name**.<br>
:small_blue_diamond: Below this group, create the groups **SNMP** and **Websites**.<br>

:collision: Unfortunately, this has an inexplicable drawback: A monitor of type **Group** is treated as a normal monitor.<br>
:collision: If one monitor in a group goes into alarm mode, the parent group also goes into alarm mode.

:point_right: You probably already have **Notifications** set up that have *Default enabled* and *Apply on all existing monitors* enabled. We disable *notifications* in every group.<br>
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

I find some of the default values ​​for an SNMP monitor to be far too low. My values ​​are:<br>
:small_blue_diamond: The value of the **Heartbeat Interval** field is set to *90 seconds*.<br>
:small_blue_diamond: The value of the **Retries** field is set to *5*<br>
:small_blue_diamond: The value of the **Heartbeat Retry** field is set to *60 seconds*.<br>

:point_right:  A Monitor is checked every *90 seconds*. If an error occurs, the monitor is retested *5 times* every *60 seconds*. An alarm is triggered after *5 minutes*.<br>

### Monitoring Memory
We are interested in the available **free RAM**. We want to know what percentage is still available.

First, let's find out how much RAM the server has available: Enter `free` on the command line. The results will be given in *KB*. It could look something like this:

```
               total        used        free      shared  buff/cache   available
Mem:         3645096      983900      908224      109888     1928288     2661196
```
This is a server with 4 GB of RAM, and slightly more than 900 MB of RAM is currently free.

To enable Uptime Kuma to read the available memory using SNMP, we need to use the appropriate *Object Identifier*: **1.3.6.1.4.1.2021.4.6.0**.<br>

The official description of this OID is: **"The amount of real/physical memory currently unused or available"**.

We can try this manually right on the command line:
```
snmpget -v 2c -c <SNMPCommunity> localhost 1.3.6.1.4.1.2021.4.6.0
```

For *\<SNMPCommunity\>*, you enter the value that is stored as *community* in the file `/etc/snmp/snmp.conf`. I'm sure you're not using *public*, right?<br>
As a result, you will get something like this:
```
UCD-SNMP-MIB::memAvailReal.0 = INTEGER: 1005112 kB
```
This value should be approximately the same as the value of *free* from the previous command.

:small_blue_diamond: Now let's create a new monitor of type **SNMP**.<br>
:small_blue_diamond: You could use *Free Memory* as the **Friendly Name**.<br>
:small_blue_diamond: Enter your server's **IP address** as the **Hostname**.<br>
:small_blue_diamond: We will leave the *Port* and the *SNMP version* at their default values.<br>
:small_blue_diamond: Now enter the name of your *SNMP community* in the **Community String** field.<br>
:small_blue_diamond: Enter *1.3.6.1.4.1.2021.4.6.0* as the **OID (Object Identifier)**.<br>

We now set the condition under which an alarm is triggered. I've chosen *20%*, which can be considered a *warning* but not yet a *critical* level.

:small_blue_diamond: Grab your calculator and calculate *20%* of the **total Memory**. Enter this value as the **Expected Value**.<br>
:small_blue_diamond: The condition is **greater than**, choose **\>**<br>

:exclamation: We store this monitor under **Monitor Group** *\<ServerName\>/SNMP*<br>

Recommendation for 20% (Warning):<br>
:small_orange_diamond:  4 GByte - Value: 729000<br>
:small_orange_diamond:  8 GByte - Value: 1531000<br>
:small_orange_diamond: 16 GByte - Value: 3143000<br>
:small_orange_diamond: 32 GByte - Value: 6400000<br>

:link: [Description for OID 1.3.6.1.4.1.2021.4.6](https://oidref.com/1.3.6.1.4.1.2021.4.6)

### Monitoring CPU Load
The appropriate OID is: **1.3.6.1.4.1.2021.10.1.3.2**.
We can check that directly on the server:
```
snmpget -v 2c -c <SNMPCommunity> localhost 1.3.6.1.4.1.2021.10.1.3.2
```

As a result, you will get something like this:
```
UCD-SNMP-MIB::laLoad.2 = STRING: 1.04
```

:small_blue_diamond: Now let's create a new monitor of type **SNMP**.<br>
:small_blue_diamond: You could use *CPU Load* as the **Friendly Name**.<br>
:small_blue_diamond: Enter your server's **IP address** as the **Hostname**.<br>
:small_blue_diamond: We will leave the *Port* and the *SNMP version* at their default values.<br>
:small_blue_diamond: Now enter the name of your *SNMP community* in the **Community String** field.<br>
:small_blue_diamond: Enter *1.3.6.1.4.1.2021.10.1.3.2* as the **OID (Object Identifier)**.<br>

We now set the condition under which an alarm is triggered. This will now be a little more complicated, because the number depends on the number of cores in the processor.

Recommendation:<br>
:small_orange_diamond: 2 Cores - Value: 3<br>
:small_orange_diamond: 4 Cores - Value: 5<br>
:small_orange_diamond: 6 Cores - Value: 7<br>
:small_orange_diamond: 8 Cores - Value: 10<br>
:small_orange_diamond: 16 Cores - Value: 20<br>

The rule of thumb is: number of cores + 25%, rounded up to a whole number.<br>With these values, a CPU is already operating under a fairly heavy, but manageable load.<br>

:small_blue_diamond: Enter the appropriate value as the **Expected Value**.<br>
:small_blue_diamond: The condition is **smaller than**, choose **\<**<br>

:exclamation: The monitor can now be saved.<br>

:link: [Description for OID 1.3.6.1.4.1.2021.10.1.3.2](https://oidref.com/1.3.6.1.4.1.2021.10.1.3.2)

### Monitoring systemd Services
This requires preparation on the target servers. First, we need to find out the names of the different processes we want to monitor. To do this, check which processes are running:
```
ps -e
```

Now we will edit the file `/etc/snmp/snmpd.conf` and add a few lines. I have already entered the most important and common processes that we would typically want to monitor.<br>
Add these lines to the end of the file:<br>

```
# Process checks
# -----------------------------------------------------------------------------

# Use ps -e to get running processes

# proc name x y
# x = Not more then x name running
# y = At least x name running, but less/equal y running

proc backup-tool
proc directadmin
proc httpd
proc java
proc mariadbd
proc mysqld
proc named
proc nginx
proc php-fpm
proc proftpd
proc python3
proc rspamd
proc redis-server
proc sshd
```

Now the SNMP service must be restarted:
```
systemctl restart snmpd
```



### Tips
:bulb: Creating these monitors for several servers takes time and patience. You can significantly simplify this process by cloning the respective monitor. Only the *IP address* and the *Expected Value* need to be adjusted.<br>


## Disadvantages, minor problems
:bomb: Uptime Kuma receives the SNMP data in **JSON format** and treats all values ​​as **strings**. This works fine until the values ​​are in decimal format. Then, comparisons become inaccurate. This affects, for example, the monitor for CPU load. The value might be **2.45**, which isn't suitable for string comparison. In that case, only the value **2** remains.<br>
:bomb: SNMP monitoring with Uptime Kuma is more of a nice-to-have feature and can't compete with proper SNMP monitoring. But that's acceptable for free software.<br>

----
:collision: If you find a mistake or any text is not understandable, please open an issue.

<p align="center">Made with :heart: and :coffee:</p>
