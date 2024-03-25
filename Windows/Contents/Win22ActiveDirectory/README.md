# Active Directory 2022


## Windows Server 2022

Check my YouTube Channel to see how you can Install windows server 2022 and customize the Windows server for use.

Below prerequisites that needed to proceed with this documentation:

1. Change Computer Name 
2. Adjust Date/Time and Timezone 
3. Set Static IP 
4. Available Disk Space on Server

> Note: If the server is on a Virtualization Environment you need to install the integrated tools on it for example vmware has VMware Tools, vbox has Guest Additions CD Image .. and so on


## Step 1: Open Server Manager 

From Dashboard select option **2 Add roles and features** 

![](attachments/Pasted%20image%2020220802132324.png)

## Step 2: Add Role and Features 

Click **Next**

![](attachments/Pasted%20image%2020220802132419.png)

## Step 3: Installation Type 

Select **Role-based or feature-based installation** then click **Next**  :

![](attachments/Pasted%20image%2020220802132452.png)

## Step 4: Server Selection

Select server you want to make it as and AD DC then Click **Next**

![](attachments/Pasted%20image%2020220802132543.png)

## Step 5: Server Roles:

Active Directory needs a DNS server to work with, so we will install both them together so select first the **Active Directory Domain Services** then next select **DNS Server**: 

![](attachments/Pasted%20image%2020220802132643.png)

## Step 6: Add Features

Once you select **Active Directory Domain Services** this window will appear for you to confirm the feature to add click **Add Features**:

![](attachments/Pasted%20image%2020220802132751.png)

Once you select **DNS Server** this window will appear for you to confirm the feature to add click **Add Features**:

![](attachments/Pasted%20image%2020220802132837.png)

## Step 7 Select Features 

Click **Next**:

![](attachments/Pasted%20image%2020220802132933.png)

## Step 8: AD DS (Domain Service )

Click **Next**:

![](attachments/Pasted%20image%2020220802133016.png)

## Step 9: DNS Server

Click **Next**:

![](attachments/Pasted%20image%2020220802133144.png)


## Step 10: Confirm your Selections

Click **Install**:

![](attachments/Pasted%20image%2020220802133226.png)

The Installation is starting

![](attachments/Pasted%20image%2020220802133256.png)

**Don't Close** this window once the installation finish check the **Step 11**:

![](attachments/Pasted%20image%2020220802134319.png)

## Step 11: Promote to DC 

Click link **Promote the server to domain controller** as shown in below screenshot:

![](attachments/Pasted%20image%2020220802134532.png)

Or If you closed the installation windows you can find this link in the **Server Manager** Task Details (Flag Icon):

![](attachments/Pasted%20image%2020220802134719.png)

Select **Add a new forest** and give a root domain name that you wish, mine is **icyb3rlab.com** :

![](attachments/Pasted%20image%2020220802135315.png)

Set a password then click **Next**:

![](attachments/Pasted%20image%2020220802135543.png)

Click **Next** : 

![](attachments/Pasted%20image%2020220802135700.png)

Click **Next**:

![](attachments/Pasted%20image%2020220802135754.png)

Click **Next**:

![](attachments/Pasted%20image%2020220802135847.png)

Click **Next**:

![](attachments/Pasted%20image%2020220802135916.png)

Click **Install** if the Prerequisites check success:

![](attachments/Pasted%20image%2020220802140118.png)

## Step 12: DNS Reverse Lookup Zone

We can create reverse DNS by below steps, Reverse DNS allow us to query the DNS using IP to get the Domain Name.

![](attachments/Pasted%20image%2020220802142100.png)

Right Click on **Reverse Lookup Zones** then **New Zone**

![](attachments/Pasted%20image%2020220802142347.png)

![](attachments/Pasted%20image%2020220802142403.png)

![](attachments/Pasted%20image%2020220802142451.png)

![](attachments/Pasted%20image%2020220802142519.png)

![](attachments/Pasted%20image%2020220802142559.png)

Here you can Enter the Network ID you want the DNS to Lookup up For:

![](attachments/Pasted%20image%2020220802142740.png)

![](attachments/Pasted%20image%2020220802142851.png)

![](attachments/Pasted%20image%2020220802142914.png)

![](attachments/Pasted%20image%2020220802143047.png)

## Step 13: Add DNS IP to Server

![](attachments/Pasted%20image%2020220802143421.png)

## Step 13: Create AD Objects 

![](attachments/Pasted%20image%2020220802143549.png)

To Add organization unit (OU) click on the Icon like in the below screenshot:

![](attachments/Pasted%20image%2020220802143737.png)

![](attachments/Pasted%20image%2020220802143846.png)

Add User to that (OU):

![](attachments/Pasted%20image%2020220802144016.png)


![](attachments/Pasted%20image%2020220802144125.png)

![](attachments/Pasted%20image%2020220802144310.png)

![](attachments/Pasted%20image%2020220802144349.png)

![](attachments/Pasted%20image%2020220802144427.png)


