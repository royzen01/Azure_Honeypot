# Azure Honeypot + Sentinel

## Decription

For this lab, I wanted to get some hands-on experience using Azure and Sentinel, so I decided to create VM and open it up to the world to make it vulnerable to attacks.

This lab will utilize Azure to host a Windows 10 VM. I will also create a Log Analytics workspace which will retrieve logs from the victim machine. Finally, I will setup Sentinel to retrieve logs from the workspace and plot on it on a live map to show the origin of the different attacks.

## Skills in practice

|      💾      |            💾       |      💾    |   
|--------------|----------------------|---------|
| Azure         | Powershell Scripting | KQL/SQL |   
| Sentinel      |  Automation          |   RDP    |   
| Event Viewer  | Windows Firewall     |     |   |


## Creating the VM

* **To start off the lab I created an Azure VM running Windows 10. This will be our honeypot so I named it `honeypot-vm`.**

> ![HP-1](https://github.com/royzen01/Azure_Honeypot/assets/13005742/ce222c3e-7f14-4e59-907d-be8adcf318d6)

* **During the VM creation process I created a new NIC security group labeled `ANY_IN`. This will put the client in that vulnerable state as I previously mentioned by allowing all incoming/outgoing traffic.**

> ![HP-2](https://github.com/royzen01/Azure_Honeypot/assets/13005742/09be557a-df47-4dc2-b001-3f1d18eb234d)

> ![HP-3](https://github.com/royzen01/Azure_Honeypot/assets/13005742/680f29ad-2d8b-40d2-a85d-87eef38992d5)

<br>

## Creating a Log Analytics workspace

* **Next, I created the Log Analytics workspace and named it `law-honeypot`.**

> ![HP-4](https://github.com/royzen01/Azure_Honeypot/assets/13005742/07b8a7ec-6e90-4263-9a33-566c883980a9)

* **Once created, I turned on Defender for the honeypot workspace**

> ![HP-5](https://github.com/royzen01/Azure_Honeypot/assets/13005742/0ec17265-d5db-47b2-b30e-66c6f6f67ac2)

* **Then I navigated to `Data collection` and made sure to select `All Events` so that we could collect all the Windows security events.**

> ![HP-6](https://github.com/royzen01/Azure_Honeypot/assets/13005742/00818f9e-ce53-447e-b6b7-90c8bf78a989)

* **I then connected the law-honeypot to the honeypot-vm to collect its logs**

> ![HP-7](https://github.com/royzen01/Azure_Honeypot/assets/13005742/24b7c98f-15d5-4b9a-aa07-945799b8865b)

<br>

## Connecting to VM using RDP

* **For the next part of the lab I had to connect to the VM using RDP.**
* **While inputting the credentials I purposely typed in the wrong password. This should hopefully create some events that we can investigate using `Event Viewer` later on.**

> ![HP-9](https://github.com/royzen01/Azure_Honeypot/assets/13005742/254b8e99-cf01-4f2d-a46c-1f42f6cbdea1)

> ![HP-10](https://github.com/royzen01/Azure_Honeypot/assets/13005742/c2839532-0dee-4978-86c0-14ba7fdd0e91)

* **Once I confirmed the VM was up and running I tried to ping it from my local machine**

> ![HP-11](https://github.com/royzen01/Azure_Honeypot/assets/13005742/93e3c6e7-0ef0-4f37-bdf6-cd938974459f)

* **As expected, the pings failed due to the Windows firewall.**
* **FOR THE PURPOSES OF THIS LAB, I will be turning off the firewall. It goes without saying this should not be done in a real working environment**
* **I turned off the `Domain`, `Private`, and `Public` profile firewalls.**

> ![HP-12](https://github.com/royzen01/Azure_Honeypot/assets/13005742/383c0600-c808-4dce-b604-df7d33e6669a)

* **Then I tried to ping the machine again. This time I was able to reach the client.**

> ![HP-13](https://github.com/royzen01/Azure_Honeypot/assets/13005742/a3cb838c-b322-4755-b02d-8c97bbaa0c19)

<br>

## Collecting logs from Event Viewer

* **In this section I collect the logs that would later be parsed into Log Analytics workspace. I did this through a Powershell Script.**
* **The purpose of the script in the following image is to query `Event Viewer` for failed RDP events. The event ID we are looking for is `4625`.**
* **Then, the script takes the source IP Address associated with the event, and parses it into the `ipgeolocation.io` API.**
* **The API provides information associated with the IP address such as `latitude`, `longitude`, and `country`.**

> ![HP-14](https://github.com/royzen01/Azure_Honeypot/assets/13005742/f9e8747a-54ea-4935-ae8b-3e925f7e1325)

* **A section of the script prints out new events into Powershell. In the following image, you can see the attempts I purposedly failed in the previous section**

> ![HP-15](https://github.com/royzen01/Azure_Honeypot/assets/13005742/ccdb5dc2-5e31-42fb-bd8f-fa5d4ab1710c)

* **The key part of the script is the following. It will take failed RDP logon requests and copy them onto a notepad file called `failed_rdp.log`.**
* **In the next section I will configure Log Analytics workspace to retrieve this log file to use it with Sentinel**

> ![HP-16](https://github.com/royzen01/Azure_Honeypot/assets/13005742/cd98ec8c-5c5f-412a-942c-0c85d87ce038)

<br>

## Parsing failed_rdp.log into workspace

* **Back in Azure, I created a custom log table for law-honeypot and provided the location of failed_rdp.log on the VM.**
* **I named the custom log `FAILED_RDP_WITH_GEO_CL`**

> ![HP-17](https://github.com/royzen01/Azure_Honeypot/assets/13005742/60d58496-ae1b-4911-9f34-f25b72cc78b4)

> ![HP-18](https://github.com/royzen01/Azure_Honeypot/assets/13005742/46a8dffb-00bb-4e70-94de-f2a9a7e6b47f)

* **Then, I tested to make sure law-honeypot was properly retrieving logs from the device. The following are from Event Viewer**

> ![HP-19](https://github.com/royzen01/Azure_Honeypot/assets/13005742/69a95867-f9e6-42f5-a2c8-9006e4b9c6a4)

* **Then, I checked to make sure it was retrieving the custom logs from failed_rdp.log**
* **Within workspace, the failed_rdp logs were being aggregated into the FAILED_RDP_WITH_GEO_CL log table**
* **I ran this table to see if the logs populated**

> ![HP-20](https://github.com/royzen01/Azure_Honeypot/assets/13005742/a92d0be9-f03a-4006-b1c1-aea99cc50458)

<br>

## Parsing logs into Sentinel

* **I then moved to create a new instance of Sentinel and connected it to law-honeypot**

> ![HP-22](https://github.com/royzen01/Azure_Honeypot/assets/13005742/eb9f04b0-ebc8-4722-b090-fa89fa97b3a8)

* **So at this stage we have collected the logs from Event Viewer using a Powershell script and copied them into a notepad file named `failed_rdp.log`.**
* **We then created a custom log table within `law-honeypot` labeled `FAILED_RDP_WITH_GEO_CL` which will retrieve and store the file.**
* **However, now we need to provide `Sentinel` with a usable format for the newly gathered data. The following query will separate the `RawData` using `KQL`**

> ![HP-22](https://github.com/royzen01/Azure_Honeypot/assets/13005742/fb26d06f-d3f1-4d26-9bca-93ead692c392)

* **Once the data is properly formatted we are able to create a new workbook within Sentinel and use the query to provide the data**
* **For visualization of where the attacks were coming from I used the latitude and longitude and plotted it on a map`**
* **Here are the end results:**

> ![HP-24](https://github.com/royzen01/Azure_Honeypot/assets/13005742/ef71c4a1-f350-43f8-ac65-230e8b6260a6)

* **After a few hours our VM was getting bombarded with brute force attacks from all over.**

> ![HP-25](https://github.com/royzen01/Azure_Honeypot/assets/13005742/b32b03f9-8eaf-4b7a-95a9-c67590133695)



