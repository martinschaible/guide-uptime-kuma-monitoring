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

:point_right: With this configuration, a ping alert will be triggered if no response occurs within 5 minutes.<br>
:point_right: This allows, for example, a server restart without triggering any alerts.<br>

:exclamation: We save the ping monitor directly in the server's group.

## Monitoring Websites
From a technical perspective, what kind of websites should be monitored? The database could fail, the web server could malfunction, or the software could have a problem.
I'm choosing one website per server, each with either an online store or a CMS like WordPress. Then I monitor the websites that are likely to be important.

:small_blue_diamond: Now create a monitor of type **HTTP(s) - Keyword**.<br>
:small_blue_diamond: I usually use the domain name as the **Friendly Name**, i.e., *mydomain.ch*.<br>
:small_blue_diamond: In the **URL** field, you enter the full address of the website, i.e., *https://www.mydomain.ch*.<br>
:small_blue_diamond: In the **Heartbeat Interval** field, I set the value to *300 seconds*.<br>
:small_blue_diamond: I leave the **Retries** value at *1* and the **Heartbeat Retry** value at *60*.<br>

## Monitoring with SNMP 
