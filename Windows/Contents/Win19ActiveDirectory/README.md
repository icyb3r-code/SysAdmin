# Active Directory 2019
Active Directory is A Microsoft product that can be installed as a windows feature. This document will guide you, to how you can install AD on windows Server 2019 Standard Edition. 

## Prerequisites
* Administrator Privilege 
* Set Static IP 
* Set Timezone and correct the time and Date.
* Set Proper Server host name, according to company naming standard.
* Add Host-name to Host file.

## Installation Steps:
## Index

	1- Windows Server 2019 Installation
	2- Prerequisites Preperation
	3- Install Active Direcotry Domain Service
	4- Promote Server as Domain Controller


### 1- OS installation


Below the steps of Install Windows server 2019 Standard on VMware:


### 2- Prerequisites

* Check the User is system user account have administrator privilege:

  ![image-20220425205139577](attachments/image-20220425205139577.png)

* Set TimeZone to your location, correct the time and date if needed .

  ![image-20220425205559409](attachments/image-20220425205559409.png)

* Check the IP address and Assign a Static IP address to the server:

  ![image-20220425205743769](attachments/image-20220425205743769.png)

* Rename the Computer or server Name to standard name:

  ![image-20220425211537205](attachments/image-20220425211537205.png)

  ![image-20220425211655131](attachments/image-20220425211655131.png)

* Add The Name to Host file is optional.

* Reboot.

  ![image-20220425211730758](attachments/image-20220425211730758.png)

### 3- Install AD 

Start Server Manager, then move to Dashboard and Click on **Add roles and features**:

![image-20220425221445390](attachments/image-20220425221445390.png)

New Window will pop up click on next 

![image-20220425221704175](attachments/image-20220425221704175.png)

Click Next 

![image-20220425221755106](attachments/image-20220425221755106.png)

Select Option same as below screen-shot then Click next:

![image-20220425221847244](attachments/image-20220425221847244.png)

Select **Active Directory Domain Controller**:

![image-20220425222034647](attachments/image-20220425222034647.png)

Click **Add Features**:

![image-20220425222117892](attachments/image-20220425222117892.png)



Select **DNS**:

![image-20220425222233039](attachments/image-20220425222233039.png)

Click **Add Features**:

![image-20220425222326785](attachments/image-20220425222326785.png)

Click Next:

![image-20220425222422813](attachments/image-20220425222422813.png)

Click Next:

![image-20220425222509802](attachments/image-20220425222509802.png)

Click Next:

![image-20220425222559421](attachments/image-20220425222559421.png)

Select **Restart the destination server automatically  if required**:

![image-20220425222800257](attachments/image-20220425222800257.png)

Monitor the Installation :

![image-20220425222907479](attachments/image-20220425222907479.png)



Don't Close the windows if the installation finish successfully:

![image-20220425223045142](attachments/image-20220425223045142.png)





### 4- Promote Server as DC



If the installation finished successfully, click on the link **Promote this server to a domain controller**:

![image-20220425223154928](attachments/image-20220425223154928.png)

Select **Add a new forest** set your preferred domain name then click **Next**:

![image-20220425223448857](attachments/image-20220425223448857.png)

Set a **DSRM Password**:

![image-20220425223757625](attachments/image-20220425223757625.png)

Click **Next**:

![image-20220425223837622](attachments/image-20220425223837622.png)

Wait little bit and then Click **Next**:

![image-20220425223948645](attachments/image-20220425223948645.png)

Click **Next**:

![image-20220425224027591](attachments/image-20220425224027591.png)

Click **Next**:

![image-20220425224116142](attachments/image-20220425224116142.png)

If the prerequisites checks passed successfully, the click **Next**:

![image-20220425224251527](attachments/image-20220425224251527.png)

Once the installation finished it will ask you for restart Click **Close**:

![image-20220425224509258](attachments/image-20220425224509258.png)



### 4- Configure DNS



![image-20220426032723359](attachments/image-20220426032723359.png)



![image-20220426032828624](attachments/image-20220426032828624.png)



![image-20220426032907448](attachments/image-20220426032907448.png)



![image-20220426033021751](attachments/image-20220426033021751.png)



![image-20220426033429172](attachments/image-20220426033429172.png)



![image-20220426033523245](attachments/image-20220426033523245.png)



![image-20220426033639746](attachments/image-20220426033639746.png)



![image-20220426033716301](attachments/image-20220426033716301.png)



![image-20220426033739264](attachments/image-20220426033739264.png)



![image-20220426033816482](attachments/image-20220426033816482.png)

To Add DNS Record 

![image-20220426034327213](attachments/image-20220426034327213.png)



![image-20220426034444386](attachments/image-20220426034444386.png)



![image-20220426034601334](attachments/image-20220426034601334.png)



![image-20220426034711587](attachments/image-20220426034711587.png)



### 6- Add OU

Click on the tools in top right cornar, then select **Active Directory users and Computers**:

![image-20220426065009261](attachments/image-20220426065009261.png)



Add new OU Organization Unit: 

![image-20220426065320603](attachments/image-20220426065320603.png)

![image-20220426065411891](attachments/image-20220426065411891.png)

Add User to the OU:

![image-20220426065533729](attachments/image-20220426065533729.png)

Insert the username as shown in below screen shot:

![image-20220426065702254](attachments/image-20220426065702254.png)

Add password to the user account, it will be used for login, Then Click next:

![image-20220426070123919](attachments/image-20220426070123919.png)

You should have the user added under the dbms OU:

![image-20220426070453391](attachments/image-20220426070453391.png)


