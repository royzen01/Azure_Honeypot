# Azure Honeypot + Sentinel

## Decription

For this lab, I wanted to get some hands-on experience using Azure and Sentinel, so I decided to create VM and open it up to the world to make it vulnerable to attacks.

This lab will utilize Azure to host a Windows 10 VM. I will also create a Log Analytics workspace which will retrieve logs from the victim machine. Finally, I will setup Sentinel to retrieve logs from the workspace and plot on it on a live map to show the origin of the different attacks.

## Skills in practice

|      -       |            -         |   |   |
|--------------|----------------------|---|---|
| Sentinel     | Powershell Scripting |   |   |
| Event Viewer | RDP                  |   |   |
| Automation   |                      |   |   |


## Step 1 - Creating the VM

* **To start off the lab I created an Azure VM running Windows 10. This will be our honeypot so I named it `honeypot-vm`.**

![HP-1](https://github.com/royzen01/Azure_Honeypot/assets/13005742/ce222c3e-7f14-4e59-907d-be8adcf318d6)

* **During the VM creation process I created a new NIC security group labeled `ANY_IN`. This will put the client in that vulnerable state as I previously mentioned by allowing all incoming/outgoing traffic.**

![HP-2](https://github.com/royzen01/Azure_Honeypot/assets/13005742/09be557a-df47-4dc2-b001-3f1d18eb234d)

![HP-3](https://github.com/royzen01/Azure_Honeypot/assets/13005742/680f29ad-2d8b-40d2-a85d-87eef38992d5)

## Step 2 - Creating a Log Analytics workspace

* **Next, I created the Log Analytics workspace and named it `law-honeypot`.**

![HP-4](https://github.com/royzen01/Azure_Honeypot/assets/13005742/07b8a7ec-6e90-4263-9a33-566c883980a9)

* **Once created, I turned on Defender for the honeypot workspace**

![HP-5](https://github.com/royzen01/Azure_Honeypot/assets/13005742/0ec17265-d5db-47b2-b30e-66c6f6f67ac2)

* **Then I navigated to `Data collection` and made sure to select `All Events` so that we could collect all the Windows security events.**

![HP-6](https://github.com/royzen01/Azure_Honeypot/assets/13005742/00818f9e-ce53-447e-b6b7-90c8bf78a989)

* **I then connected the law-honeypot to the honeypot-vm to collect its logs**

![HP-7](https://github.com/royzen01/Azure_Honeypot/assets/13005742/24b7c98f-15d5-4b9a-aa07-945799b8865b)

## Connecting to VM using RDP

* **For the next part of the lab I had to connect to the VM using RDP.**
* **While inputting the credentials I purposely typed in the wrong password. This should hopefully create some events that we can investigate using `Event Viewer` later on.**

![HP-9](https://github.com/royzen01/Azure_Honeypot/assets/13005742/254b8e99-cf01-4f2d-a46c-1f42f6cbdea1)

![HP-10](https://github.com/royzen01/Azure_Honeypot/assets/13005742/c2839532-0dee-4978-86c0-14ba7fdd0e91)

* **Once I confirmed the VM was up and running I tried to ping it from my local machine**

![HP-11](https://github.com/royzen01/Azure_Honeypot/assets/13005742/93e3c6e7-0ef0-4f37-bdf6-cd938974459f)

* **As expected, the pings failed due to the Windows firewall.**
* **FOR THE PURPOSES OF THIS LAB, I will be turning off the firewall. It goes without saying this should not be done in a real working environment**
* **I turned off the `Domain`, `Private`, and `Public` profile firewalls.**

![HP-12](https://github.com/royzen01/Azure_Honeypot/assets/13005742/383c0600-c808-4dce-b604-df7d33e6669a)

* **Then I tried to ping the machine again**

![HP-13](https://github.com/royzen01/Azure_Honeypot/assets/13005742/a3cb838c-b322-4755-b02d-8c97bbaa0c19)


