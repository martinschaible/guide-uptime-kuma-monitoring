## Uptime Kuma is here!
In this article, you'll learn how to monitor websites and servers using **Uptime Kuma**.

Monitoring a server using SNMP isn't exactly modern, and specialized monitoring software solves this problem with agents that are installed on the target system.

Smaller companies with just a few servers and limited budgets often prefer to avoid expensive and overly complex monitoring software.

:collision: To be able to react quickly to errors and failures, a few checks are sufficient, which can be accomplished with SNMP.

## Basics
Before we begin monitoring, let's start with some organizational steps:

:small_blue_diamond: Create a new **Monitor** of type **Group** for each server. Use the server name as the **Friendly Name**<br>
:small_blue_diamond: Below this group, create the groups **SNMP** and **Websites**.<br>

:point_right: You can leave all other values ​​in the group as they are. At first, I thought these values ​​would be inherited by new objects. Unfortunately, that's not the case.

## Monitoring Websites

## Monitoring with SNMP 
