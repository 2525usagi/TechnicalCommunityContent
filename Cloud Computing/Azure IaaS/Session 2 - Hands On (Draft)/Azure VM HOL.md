<a name="HOLTitle"></a>
# Working with Azure Virtual Machines #

---

<a name="Overview"></a>
## Overview ##

The Azure Virtual Machine service provides access to a vast array of hardware, networking, operating system (OS), and software combinations that you can use to support custom workloads in the cloud.  Virtual Machines (VMs) fall into the Infrastructure-as-a-Service (IaaS) model of cloud computing, abstracting away the management of the physical hardware while allowing access to the computing resources. This arrangement affords users flexibility and control, while at the same time offloading the responsibility for maintenance and upkeep tasks such as applying patches to the operating system. You can learn more about IaaS and other cloud computing models [here](https://en.wikipedia.org/wiki/Cloud_computing#Infrastructure_as_a_service_.28IaaS.29).

It has been widely reported that Microsoft has gone through profound changes in the past several years. One facet of these changes is an embrace of both Linux and Open Source technologies.  The VMs that you create and manage in Microsoft Azure are not limited to just Windows Server images, but also include a wide selection of Linux VMs, including Ubuntu, Red Hat, Debian, SUSE, Oracle, and CentOS distributions. If those do not meet your needs, you can also follow [these instructions](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-classic-create-upload-vhd) to upload your own VM image.
  
In this lab, you will learn how you can use the gallery of proconfigured VM images in the Azure Portal to provision an Ubuntu Linux VM. You will then see how you can use a Secure Shell (SSH) terminal to log in to your VM and perform several installation and customization tasks in order to allow it act as a simple web server. You will also see how you can work with Network Security Group rules to govern the network traffic to and from your VM.  

<a name="Objectives"></a>
### Objectives ###

In this hands-on lab, you will learn how to:

- Provision a Linux virtual machine in Azure using the Azure Marketplace images
- Connect to the virtual machine using SSH
- Install a web server on the virtual machine
- Define a network security group rule to allow incoming network traffic on the HTTP port
- Stop the virtual machine to suspend any further usage charges
- Use the Azure Resource Manager to delete the virtual machine and its related resources

<a name="Prerequisites"></a>
### Prerequisites ###

The following are required to complete this hands-on lab:

- An active Microsoft Azure subscription, or [sign up for a free trial](http://aka.ms/WATK-FreeTrial)
- [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) (Windows users only)

<a name="Exercises"></a>
## Exercises ##

This hands-on lab includes the following exercises:

- [Exercise 1: Provision a virtual machine](#Exercise1)
- [Exercise 2: Set up a Web server on the virtual machine](#Exercise2)
- [Exercise 3: Configure access to the HTTP port](#Exercise3)
- [Exercise 4: Suspend the virtual machine](#Exercise4)
- [Exercise 5: Delete the lab resources](#Exercise5)

Estimated time to complete this lab: **60** minutes.

<a name="Exercise1"></a>
## Exercise 1: Provision a virtual machine ##

The [Azure Portal](https://portal.azure.com) allows you to perform many of the operations required to create and manage virtual machines. It also provides access to a browsable marketplace that you can use to find the virtual machine images you want to deploy. In this exercise, you'll use the Azure Portal to find a virtual machine image in the Azure Marketplace and use it to create a new virtual machine instance.

1. Go to the [Azure Portal](https://portal.azure.com/). If you are asked to sign in, do so using your Microsoft account.

1. Click **+ New** in the ribbon on the left. Then select **Compute**, followed by **See All**.

    ![Adding a virtual machine](Images/provision-newcompute.png)

     _Adding a virtual machine_

1. Type "ubuntu" (without quotation marks) into the search box and press **Enter**. Then click **Ubuntu Server 16.04 LTS**.

    ![Selecting an Ubuntu image](Images/provision-selectubuntu.png)

     _Selecting an Ubuntu image_

1. Make sure that **Resource Manager** is selected under **Select a deployment model**, and then click the **Create** button.

    ![Selecting a deployment model](Images/provision-create.png)

     _Selecting a deployment model_

1. In the "Basics" blade, enter "VMLab" (without quotation marks) as the virtual-machine name. Select **SSD** as the disk type and enter "azureuser" as the **User name**. Select **Password** as the authentication type and enter "Azure4Research" as the password. Select **Create new** under **Resource group** and enter "VMLabResourceGroup" as the resource-group name. Under **Location**, select the location nearest you. Then click **OK**.

    ![Specifying basic VM settings](Images/provision-settings-basics.png)

     _Specifying basic VM settings_
 
1. In the "Choose a size" blade, select **DS1_V2 Standard** and click the **Select** button.

    ![Selecting a virtual machine size](Images/provision-settings-choosesize.png)

     _Selecting a virtual machine size_

1. Click the **OK** button at the bottom of the "Settings" blade to accept the default values.

1. Click the **OK** button at the bottom of the "Summary" blade to begin provisioning the virtual machine.

    ![Reviewing the virtual machine settings](Images/provision-settings-reviewsummary.png)

     _Reviewing the virtual machine settings_

1. When the provisioning process completes after a couple of minutes, a blade for the VM should automatically appear. Click **VMLabResourceGroup** to open the resource group that contains all of the resources related to this VM.     

    ![Opening the VM's resource group](Images/provision-vmlab-blade.png)

     _Opening the VM's resource group_

1. Examine the list of resources that were provisioned along with the virtual machine. Key resources include:

	- The virtual machine
	- The network interface (NIC) used by the VM for communication
	- A network security group containing port-access restrictions that govern communication types through the network interface
	- The VM's public IP address
	- The virtual network to which the VM belongs
	- Two storage accounts: one to store disk images for the VM, and one to store diagnostic information

    ![The VM and associated resources](Images/provision-vmlab-resourcegroup.png)

     _The VM and associated resources_

1. Close the "VMLabResourceGroup" blade to return to the blade for the VM.

The VM is provisioned and ready to be used. The next step is to do something with it. But first, you need to connect to it via SSH.

<a name="Exercise2"></a>
## Exercise 2: Set up a Web server on the virtual machine

In this exercise, you will configure your virtual machine to handle web requests using the Node.js runtime.  work on the VM remotely, you will use a Secure Shell terminal application.  On Mac and Linux systems, SSH is built into the operating system and available for you to use.  On Windows, you will use the popular Windows SSH client named PuTTY.

1. The deployment template that you used to create the virtual machine created a public IP address for the virtual machine.  Open the blade for the **VMLab** virtual machine if it is not already open and click on the IP address under the **Public IP address/DNS name label** section to open the blade for the Public IP address resource.

    ![Opening the IP address blade](Images/webserver-open-ipaddress.png)

    _Opening the IP address blade_
 
2. In the **VMLab-ip** blade, hover your mouse over the **IP address** entry.  When the **Click to copy** icon appears, click it to copy the IP address to the clipboard.

    ![Copying the public IP address](Images/webserver-copy-ipaddress.png)

    _Copying the public IP address_

### Connecting with MacOS/Linux ###
Perform these steps to establish a connection to the virtual machine if you are using MacOS or Linux.  If you're running Windows, skip ahead to the *Connecting with Windows* section.

1. Open a terminal window to use to log into the virtual machine. Execute the following command.  If you chose a different VM administrator name than "_azureuser_" in Exercise 1, replace _azureuser_ with the name you chose.  Replace _IP-Address_ with the IP address that you copied into the clipboard in the previous step.

    <pre>
    ssh -l <i>azureuser</i> <i>IP-Address</i>
    </pre>

	> Because this is the first time you have connected to the virtual machine, you may be prompted with a warning dialog asking if you trust this host. Since the host is one you created, type "yes" (without the quotes).

	When prompted, enter the password "_Azure4Research_" (without the quotes) or the value you provided in Exercise 1 if you chose to use a differnet password.

### Connecting with Windows ###
If you already installed PuTTY, [download the installer MSI file](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) and install it now.

1. Start PuTTY and paste the IP Address into the **Host Name (or IP address)** field. Then click the **Open** button to initiate a Secure Shell (SSH) connection.

	> Because this is the first time you have connected to the virtual machine, you may be prompted with a warning dialog asking if you trust this host. Since the host is one you created, click **Yes**.

    ![Connecting with PuTTY](Images/webserver-putty-connect.png)

    _Connecting with PuTTY_

1. A PuTTY terminal window will appear and you will be prompted to **login as**. Log in with the the VM administrator user name ("azureuser") and password ("Azure4Research") you previously entered in [Exercise 1](#Exercise1).

### Configuring the virtual machine ###

1. In the SSH terminal, execute the commands below to update the package lists, install the Node.js runtime, install the NPM package manager, and update the internal firewall to allow port 80 communication on the virtual machine. If you are prompted to continue, type "Y" (without the quotes) to indicate "yes".
    <pre>
    sudo apt-get update
	</pre>

    <pre>
    sudo apt-get install nodejs
	</pre>

	<pre>
	sudo apt-get install npm
    </pre>

    <pre>
    sudo ufw allow 80
    </pre>

1. Edit the node content file in the SSH terminal by typing the following command:
	<pre>
    vi labvm.js
	</pre>

1. Press the "i" (without quotes) key to enter _Insert Mode_ in VI.  The bottom left corner of your screen should display the text

	> -- INSERT --

1. Paste the following text into the VI editor:
	<pre>
	var http = require('http');
	
	var server = http.createServer(function(req, res) {
	  res.writeHead(200);
	  res.end('Hello Azure VM Lab!');
	});
	server.listen(80);
	</pre>
 
1. To finish editing, press the Escape (ESC) key on your keyboard, then the following key sequence to save and exit.
	- Esc (escape key)
	- : (colon)
	- wq
	- Enter key

1. Enter the following command in your SSH terminal to run the the node program you just wrote.  This program will listen for web request on port 80 returning a fixed block of text for each request.
	<pre>
	sudo nodejs labvm.js 
	</pre>
  
<a name="Exercise3"></a>
## Exercise 3: Configure access to the HTTP port

Even though you configured the firewall in the VM to allow network traffic on port 80, there is still one more step you have to perform in order to be able to serve web content.  That is because the _Virtual Network_ (VNET) that your machine resides in includes a _Network Security Group_ (NSG).  The NSG includes rules that define the kinds of inbound and outbound communications that are allowed, and you have yett o tell it that it should allow communication on port 80, the HTTP port.  In this exercise, you will configure a rule that will allow external traffic to reach your virtual machine's _Network Interface Card_ (NIC).

1. In the Azure Portal, open the blade for your lab VM if it is not already open.  Click on **Network Interfaces** to bring up the blade for your VM's NIC, then click on the NIC for your VM in the list that is shown.

    ![Open the NIC blade for the VM](Images/portconfig-open-nic.png)

    _Open the NIC blade for the VM_

1. In the **Network Interface** blade, click on the **Network security group** entry, and then on the name of the network security group to bring up its blade.

    ![Open the network security group](Images/portconfig-open-nsg.png)

    _Open the network security group_

1. Click on **Inbound security rules** and then on **Add** to add a new rule.

    ![Add a new inbound rule](Images/portconfig-add-innboundrule.png)

    _Add a new inbound rule_

1. Enter the following information for the new rule:
	- Enter "Lab-VM-HTTP" (without the quotes) in the **Name** box
	- Leave the **Priority** box set to **1010**
	- Leave the **Source** box set to **Any**
	- Under **Service**, select **HTTP**
	- Leave the **Action** selection set to **Allow**
	
	Press the **OK** button to create the rule.

    ![Define the new inbound rule](Images/portconfig-set-inboundrule.png)

    _Define the new inbound rule_

	You may need to wait a minute or two for the new rule to be created.

1. Open a new browser window and enter the IP address of your VM into the address bar.  It should show the content you entered in the Node program in the previous exercise.

    ![Browse the web page](Images/portconfig-final-sitedisplay.png)

    _Browse the web page_

<a name="Exercise4"></a>
## Exercise 4: Suspend the virtual machine

When virtual machines are running, you are being charged — even if the VM is idle. Therefore, it is advisable to stop virtual machines when they are not in use. You will still be charged for storage, but that cost is typically insignificant compared to the cost of an active VM. The Azure Portal makes it easy to stop virtual machines. VMs that you stop are easily started again later so you can pick up right where you left off.

1. In the Azure Portal, open the blade for your lab VM if it is not already open.  

1. Click the **Stop** button to stop the virtual machine.

    ![Stopping the virtual machine](Images/suspend-stop-vm.png)

    _Stopping the virtual machine_

You can stop and start virtual machines in the Azure portal, but if you have a lot of VMs, that's not very efficient. In the real world, you might prefer to use an Azure CLI or PowerShell script to enumerate all of the VMs in a resource group and start or stop them all. For more information on scripting the Azure CLI, see the section entitled "How to script the Azure CLI for Mac, Linux, and Windows" in [Install and Configure the Azure CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli/). If you prefer visual tools to command-line tools, you can use [Azure Automation](https://azure.microsoft.com/en-us/services/automation/) to automate VM operations.

<a name="Exercise5"></a>
## Exercise 5: Delete the lab resources

Resource groups are a useful feature of Azure because they simplify the task of managing related resources. One of the most practical reasons to use resource groups is that deleting a resource group deletes all of the resources it contains. Rather than delete those resources one by one, you can delete them all at once.

In this exercise, you'll delete the resource group created in [Exercise 1](#Exercise1) when you provisioned the virtual machine. Deleting the resource group deletes everything in it and prevents any further charges from being incurred for it.

1. In the Azure Portal, open the blade for the "VMLabResourceGroup" resource group that holds the virtual machine. Then click the **Delete** button at the top of the blade.

	![Deleting a resource group](Images/delete-delete-resourcegroup.png)

	_Deleting a resource group_

1. For safety, you are required to type in the resource group's name. (Once deleted, a resource group cannot be recovered.) Type the name of the resource group. Then click the **Delete** button to remove all traces of this lab from your account.

After a few minutes, the virtual machine and all of its resources will be deleted. Billing stops when you click the **Delete** button, so you're not charged for the time required to delete the virtual machine. Similarly, bulling doesn't start until a virtual machine has been fully and successfully deployed.

## Summary ##

In this hands-on lab, you learned how to:

- Provision a Linux virtual machine in Azure using the Azure Marketplace images
- Connect to the virtual machine using SSH
- Install a web server on the virtual machine
- Define a network security group rule to allow incoming network traffic on the HTTP port
- Stop the virtual machine to suspend any further usage charges
- Use the Azure Resource Manager to delete the virtual machine and its related resources

The Azure Virtual Machine service gives you a lot of powerful options for configuring and deploying virtual machines into the Cloud.  But remember, "with great power, comes great responsibility."  Although VM's allow you to the greatest options for custom configuration, you are also signing up for the related responsibility of managing the VM, especially including timely application of OS and security patches.

---

Copyright 2016 Microsoft Corporation. All rights reserved. Except where otherwise noted, these materials are licensed under the terms of the MIT License. You may use them according to the license as is most appropriate for your project. The terms of this license can be found at https://opensource.org/licenses/MIT.