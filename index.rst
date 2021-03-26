.. Documentation master file, created by
   sphinx-quickstart on Sat Mar  6 02:48:40 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.
===========================================
FreeRADIUS Agent for SafeNet Trusted Access
===========================================

.. toctree::
   :hidden:
   :maxdepth: 3
   

   index
      




Overview
^^^^^^^^
The **SafeNet agent for FreeRADIUS** extends the open-source FreeRADIUS project to offer a highly secure container-based service that enables legacy RADIUS clients such as VPN gateways and routers to perform authentication and authorization against SafeNet Authentication Service (SAS-PCE) or SafeNet Trusted Access (STA).
 
While abstracted from the customer through containerization it may be interesting to understand that the SafeNet agent for FreeRADIUS installs as a module to the open-source based FreeRADIUS server. In the SafeNet agent bundle, the FreeRADIUS server configuration has been configured on behalf of the customer to call the :abbr:`sasagent (SafeNet Authentication Service Agent)` module for supported RADIUS protocol requests.

FreeRADIUS takes in a standardized RADIUS request over UDP on port 1812 (configurable) and if the client (e.g. the VPN gateway) is authorized based on the configured RADIUS clients list, the sasagent module will forward end-user authentication and authorization to either SafeNet Authentication Service (SAS) or SafeNet Trusted Access (STA).

This 'forwarding' as well as the return decision (accept/reject) is done using SafeNet proprietary encryption over TLS (TCP port 443) facilitated through the use of a key file (the :file:`Agent.bsidkey`) as well as explicit authentication node ("auth node") authorization in the SAS or STA virtual server.
 
 .. thumbnail:: /images/freeradius/freeRADIUSArchitecture.png
   :width: 100%
   :title: Figure: High level deployment architecture using FreeRADIUS.
   :show_caption: true
|

Installation
^^^^^^^^^^^^

#. Open a terminal to the FreeRADIUS agent host
#. Install yum-utils, if not already present:
   
   ::

       sudo yum install -y yum-utils

#. Add the Docker repository:
   
   ::

       sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#. Install Docker (and Containerd) answering ‘yes’ to prompts:
   
   ::

       sudo yum install docker-ce docker-ce-cli containerd.io -y

#. Start Docker:

   ::

       systemctl start docker

#. Create a Docker service user and group: 
   
   ::

       sudo useradd freeradius

   ::

       sudo usermod -a -G docker freeradius

#. Enable Docker and Containerd to run as a service: 

   ::

       sudo systemctl enable docker.service

   ::

       sudo systemctl enable containerd.service


Configuration
^^^^^^^^^^^^^
At a high level there are two major configuration steps once the SafeNet FreeRADIUS agent is installed:

*  **Configuring FreeRADIUS**
*  **Configuring client(s)**

The first step above configures FreeRADIUS to be able to address either SAS or STA with RADIUS related requests and the second step then configures any client -that is: any system or service that will authenticate users over RADIUS, to be *authorized* to send requests to FreeRADIUS. Notably this step also involves defining the same client(s) in SAS or STA. Both topics are covered extensively below.

Configuring FreeRADIUS
======================

Download the Agent.bsidkey file
-------------------------------

To download the key file:

#. Open a browser and navigate to **SAS/STA**
#. Authenticate as operator and then navigate to the :guilabel:`Comms` tab
#. Click to expand :guilabel:`Authentication Processing` and then expand :guilabel:`Authentication Agent Settings`

   .. thumbnail:: /images/freeradius/downloadKeyFile.png
      :width: 80%
      :title: Figure: Downloading the key file from STA.

#. Click :guilabel:`Download` to generate and save the key file
#. Transfer the :file:`Agent.bsidkey` to the FreeRADIUS host using your tool of choice

.. _Running-the-configuration-script:

Running the configuration script
--------------------------------

To run the configuration script:

#. Open a terminal to the FreeRADIUS agent host
#. Run the :code:`FreeRADIUSv3.sh` by executing the following command

   ::

        sudo sh ./FreeRADIUSv3.sh

#. Press :kbd:`Enter` when prompted to start the (re)configuration

   .. code-block:: text

      Provide neccessary configurations for FreeRADIUS container to run. Press [Enter] to continue.

#. Press :kbd:`Y` to switch to HTTPS (TLS)

   .. code-block:: text
      :emphasize-lines: 3

      Default protocol is http. Change to https? Y/N
      Ensure that Y is entered if using SAS cloud
      y

#. Type the FQDN of your SAS or STA service (refer to table below for guidance) and press :kbd:`Enter`

   .. code-block:: text
      :emphasize-lines: 2
    
      Please Enter SafeNet Authentication Service IP or FQDN.
      cloud.eu.safenetid.com
      
      Validating the SAS Token Validator URL is accessible...
      The SAS Token Validator URL is accessible.


   +---------+-------------------------------------------------------------------------+
   | Zone    | FQDN                                                                    |
   +=========+=========================================================================+
   | EU      | cloud.eu.safenetid.com                                                  |
   +---------+-------------------------------------------------------------------------+
   | US      | cloud.eu.safenetid.com                                                  |
   +---------+-------------------------------------------------------------------------+
   | Classic | agent1.safenet-inc.com                                                  |
   +---------+-------------------------------------------------------------------------+

#. Press :kbd:`N` when prompted about the API

   .. code-block:: text
      :emphasize-lines: 3
    
      Is the SAS RADIUS Client API URL accessible? Y/N
      Ensure that N is entered if using SAS cloud
      n

   .. attention::
      The RADIUS Client API is responsible for updating the RADIUS Clients from the SAS server to the FreeRADIUS server when using an on-premise deployment model with SAS-PCE or SAS-SPE. This deployment model is out of scope for this documentation.
   
#. Type the path to the :file:`Agent.bsidkey` and press :kbd:`Enter`

   .. code-block:: text
      :emphasize-lines: 2
    
      Enter complete path of Agent BSID key file.
      ./Agent.bsidkey
      Validating if Agent BSID key file exists at the given path...
      Agent BSID key file exists.

#. Press :kbd:`Y` when asked about PEAP support (see note)
   
   .. code-block:: text
      :emphasize-lines: 2
    
      Do you wish to use default certificates for PEAP support? Y/N
      y

   .. note::
   
      Pressing :kbd:`Y` will use bundled self-signed certificates. These may or may not work with PEAP. If you do intend to use PEAP exit the script and generate or obtain the necessary certificates! If you do select :kbd:`N` then additional prompts will follow. These are **not** documented here. If you need guidance on these prompts please refer to official Thales product documentation.
   
#. Type the RADIUS (authentication) port number, typically **1812** and press :kbd:`Enter` 
   
   .. code-block:: text
      :emphasize-lines: 2
    
      Enter Port Number for FreeRadius container.
      1812
      The Port is accessible.

#. When prompted about UTF8 support, press :kbd:`N` (see note)
   
   .. code-block:: text
      :emphasize-lines: 2
    
      By Default the FreeRADIUS agent supports iso-8859-1 encoding. Change to UTF8? Y/N
      n

   .. note::
   
      UTF8 may be required if you expect to see national characters, e.g. user names containing **å**, **ä**, **ö** on the wire. Note though that most NAS will not support UTF8 and so supporting it in FreeRADIUS agent is not necessarily a full solution.

#. Either accept using **SYSLOG** for logging by pressing :kbd:`N` or :kbd:`Y` to use JSON written to file

   .. code-block:: text
      :emphasize-lines: 2
    
      Default log driver for FreeRADIUS container is set to 'SYSLOG'. Change to JSON-FILE? Y/N
      n

   .. note::
   
      If you do select JSON-FILE then additional prompts will follow. These are **not** documented here. If you need guidance on these prompts please refer to official Thales product documentation.


#. Finally, the script prompts for using internal (default) or external SYSLOG. Press :kbd:`N` for using local SYSLOG or :kbd:`Y` to change to an external server 

   .. code-block:: text
      :emphasize-lines: 2
    
      Do you want to use an external Syslog server? Y/N
      n
      Checking if syslog daemon is running on host machine..

   .. note::
   
      If you do select to use an external server then additional prompts will follow. These are **not** documented here. If you need guidance on these prompts please refer to official Thales product documentation.

.. thumbnail:: /images/freeradius/runConfiguration.gif
   :title: Demonstration: Running the configuration script.
   :show_caption: true
|

.. tip::
   
   If you want to automate this part of configuration for repeat use or to better support upgrading then you can do something like (based on above input as an example):
   
   ::

        printf '%s\n' Y Y cloud.eu.safenetid.com N ./Agent.bsidkey Y 1812 N N N | sh ./FreeRADIUSv3.sh
   
   In the above example, input to script prompts such as :code:`Y` (Yes) is provided, including line breaks :code:`\n` (newline). 
   If you do take this approach you have to stay on top of new prompts being introduced in later versions of the script! 


Configuring client(s)
=====================
RADIUS clients such as your NAS or test tool must be added *both* to the SafeNet FreeRADIUS agent and to SAS/STA.


FreeRADIUS
----------

To add and manipulate RADIUS clients, a SafeNet :abbr:`CRUD (Create, Read, Update and Delete)` script called :code:`Client_Updater.sh` is provided with the agent software bundle. This section describes how to manually create clients using the script. 

**Reading** (viewing), **Updating** and **Deleting** clients is not explicitly documented here, but follows the same principle, using the **V**, **U** and **D** option respectively with the script.

.. attention::
   Using the :code:`Client_Updater.sh` script is relevant **only** if you are configuring the SafeNet FreeRADIUS agent against SAS Cloud or STA. If you are configuring an on-premise version of SAS, e.g. SAS-PCE or SAS-SPE then updating clients can be automated using the RADIUS API (out of scope for this documentation).
   
To add a RADIUS client:

#. Open a terminal to the FreeRADIUS agent host
#. Run the :code:`Client_Updater.sh` by executing the following command

   ::

        sudo sh ./Client_Updater

#. When prompted, type :kbd:`C`, and press :kbd:`Enter`
#. Type the IP address of your RADIUS client and press :kbd:`Enter`
#. Type a name (any) for the RADIUS client and press :kbd:`Enter`
#. Type a shared secret (the same secret must be in your client!) and press :kbd:`Enter`

.. thumbnail:: /images/freeradius/addClient.gif
   :align: center
   :title: Demonstration: Running the client update script to add (create) a client.
   :show_caption: true
|

SAS/STA
-------
The same clients added in FreeRADIUS using the :code:`Client_Updater.sh` script must also be added to SAS or STA.

To add a RADIUS client:

#. Open a browser and navigate to **SAS/STA**
#. Authenticate as operator and then navigate to the :guilabel:`Comms` tab
#. Expand :guilabel:`Auth Nodes`
#. Click :guilabel:`Add`
#. Provide relevant attributes including name and IP address

   .. thumbnail:: /images/freeradius/authNodes.png
      :width: 80%
      :title: Figure: Configuring the RADIUS client as an Auth Node in STA.


#. Click :guilabel:`Save`


Automatic service restart
=========================
By default the SafeNet FreeRADIUS container much like other containerized services will not start automatically. To ensure the service recovers on for example a system reset event or to simply avoid having to manually start and stop the container, a Docker restart policy can be set. The below command example configures the SafeNet FreeRADIUS container to always restart. 

.. note::
   For more information about Docker restart policies, please refer to `Docker documentation <https://docs.docker.com/config/containers/start-containers-automatically/>`__ 

Configure FreeRADIUS to start automatically:
::

    sudo docker update --restart always FreeRADIUSv3


Deployment Automation
^^^^^^^^^^^^^^^^^^^^^

Youtube video here

Using Cloud-Init
================
The following is an *example* cloud-init file that can support some amount of deployment automation, namely installing and configuring Docker as well as loading the SafeNet FreeRADIUS container image.
The file below was adapted for use with Microsoft Azure (see *next* section) and uses **explicit commands** in sequence. In general, it should be possible to use additional abstraction. 

.. note::
   Step by step usage of cloud-init is currently out of scope for this documentation, but the video in the previous section provides some insight.

.. warning::
   This cloud-init is provided as an example only and should not be used in a production system without extensive review and testing.

Sample cloud-init file:

.. code-block:: yaml
   :linenos:
   
   #cloud-config
   runcmd:
   - sudo yum install -y yum-utils
   - sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   - sudo yum install docker-ce docker-ce-cli containerd.io -y
   - sudo systemctl start docker
   - sudo useradd freeradius
   - sudo usermod -a -G docker freeradius
   - sudo systemctl enable docker.service
   - sudo systemctl enable containerd.service
   - sudo wget -O /home/userName/FreeRADIUS_Agent.zip "https://softwareLocation/"
   - sudo unzip /home/userName/FreeRADIUS_Agent.zip -d /home/userName/
   - sudo cp  /home/SO/FreeRADIUS_Agent/Installers/*.sh /home/userName/
   - sudo rm /home/userName/FreeRADIUS_Agent.zip
   - sudo docker load --input /home/userName/FreeRADIUS/Installers/fragent-version.tar


Using an ARM template
=====================
The following is an *example* that extends the use of the cloud-init file shown in the previous section by referencing it from inside an :abbr:`ARM (Azure Resource Manager)` template using the :code:`customData` capability. As the naming implies, this example is only applicable to use with Microsoft Azure.

In the below example, a CentOS host is installed from Azure Marketplace and configured with Docker and a set up supporting parameters like basic firewall rules to permit RADIUS traffic as well as resource tags. To improve speed of deployment the required software and other artifacts like the cloud-init.txt file is hosted from Azure Container (blob) storage and accessed using a :abbr:`SAS (Shared Access Signature)` token.

.. note::
   Step by step usage of ARM templates is currently out of scope for this documentation, but the video in the previous section provides some insight.

.. warning::
   This ARM template is provided as an example only and should not be used in a production system without extensive review and testing.

Sample Azure Resource Manager Template:

.. code-block:: json
   :linenos:
   
   {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string",
            "defaultValue": "FreeRADIUS",
            "metadata": {
                "description": "The name of you Virtual Machine."
            }
        },
        "adminUsername": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "Username for the Virtual Machine."
            }
        },
        "authenticationType": {
            "type": "string",
            "defaultValue": "password",
            "allowedValues": [
                "sshPublicKey",
                "password"
            ],
            "metadata": {
                "description": "Type of authentication to use on the Virtual Machine. SSH key is recommended."
            }
        },
        "adminPasswordOrKey": {
            "type": "securestring",
            "metadata": {
                "description": "SSH Key or password for the Virtual Machine. SSH key is recommended."
            }
        },
        "virtualMachineSize": {
            "type": "string",
            "defaultValue": "Standard_DS1_v2",
            "metadata": {
                "description": "VM size for the host running FreeRADIUS."
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "resourceTags": {
            "type": "object",
            "defaultValue": {
                "Product": "SAS-PCE",
                "Component": "FreeRADIUS"
            }
        }
    },
    "variables": {
        "nicName": "myVMNicD",
        "addressPrefix": "10.0.0.0/16",
        "subnetName": "Subnet",
        "subnetPrefix": "10.0.0.0/24",
        "diskStorageType": "Standard_LRS",
        "publicIPAddressName": "myPublicIPD",
        "publicIPAddressType": "Dynamic",
        "virtualNetworkName": "MyVNETD",
        "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
        "linuxConfiguration": {
            "disablePasswordAuthentication": true,
            "ssh": {
                "publicKeys": [
                    {
                        "path": "[concat('/home/', parameters('adminUsername'), '/.ssh/authorized_keys')]",
                        "keyData": "[parameters('adminPasswordOrKey')]"
                    }
                ]
            }
        },
        "networkSecurityGroupName": "default-NSG"
    },
    "resources": [
        {
            "apiVersion": "2015-05-01-preview",
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[variables('publicIPAddressName')]",
            "location": "[parameters('location')]",
            "tags": "[parameters('resourceTags')]",
            "properties": {
                "publicIPAllocationMethod": "[variables('publicIPAddressType')]"
            }
        },
        {
            "comments": "Default Network Security Group for template",
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2020-05-01",
            "name": "[variables('networkSecurityGroupName')]",
            "location": "[parameters('location')]",
            "tags": "[parameters('resourceTags')]",
            "properties": {
                "securityRules": [
                    {
                        "name": "default-allow-22",
                        "properties": {
                            "priority": 1001,
                            "access": "Allow",
                            "direction": "Inbound",
                            "destinationPortRange": "22",
                            "protocol": "Tcp",
                            "sourceAddressPrefix": "*",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*"
                        }
                    },
                    {
                        "name": "default-allow-1812",
                        "properties": {
                            "priority": 1002,
                            "access": "Allow",
                            "direction": "Inbound",
                            "destinationPortRange": "1812",
                            "protocol": "Udp",
                            "sourceAddressPrefix": "*",
                            "sourcePortRange": "*",
                            "destinationAddressPrefix": "*"
                        }
                    }
                ]
            }
        },
        {
            "apiVersion": "2020-05-01",
            "type": "Microsoft.Network/virtualNetworks",
            "name": "[variables('virtualNetworkName')]",
            "location": "[parameters('location')]",
            "tags": "[parameters('resourceTags')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
            ],
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('addressPrefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[variables('subnetName')]",
                        "properties": {
                            "addressPrefix": "[variables('subnetPrefix')]",
                            "networkSecurityGroup": {
                                "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "apiVersion": "2020-05-01",
            "type": "Microsoft.Network/networkInterfaces",
            "name": "[variables('nicName')]",
            "location": "[parameters('location')]",
            "tags": "[parameters('resourceTags')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'))]",
                "[resourceId('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIPAddressName'))]"
                            },
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "apiVersion": "2019-07-01",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "[parameters('vmName')]",
            "location": "[parameters('location')]",
            "tags": "[parameters('resourceTags')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('virtualMachineSize')]"
                },
                "osProfile": {
                    "computerName": "[parameters('vmName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPasswordOrKey')]",
                    "linuxConfiguration": "[if(equals(parameters('authenticationType'), 'password'), json('null'), variables('linuxConfiguration'))]",
                    "customData": "[base64(concat('#include\nhttps://pubswedemo.blob.core.windows.net/freeradius/cloud-init.txt?sp=r&st=2021-03-16T09:17:28Z&se=2021-03-16T17:17:28Z&spr=https&sv=2020-02-10&sr=c&sig=eXJCTNEA5vK4TRTVoNJQOu19n9h%2BZSocEMBzx4CfL9Y%3D'))]"
                },
                "storageProfile": {
                    "imageReference": {
                        "publisher": "OpenLogic",
                        "offer": "CentOS",
                        "sku": "7.7",
                        "version": "latest"
                    },
                    "osDisk": {
                        "name": "[concat(parameters('vmName'),'_OSDisk')]",
                        "caching": "ReadWrite",
                        "createOption": "FromImage",
                        "managedDisk": {
                            "storageAccountType": "[variables('diskStorageType')]"
                        }
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName'))]"
                        }
                    ]
                }
            }
        }
    ]
   }
   
  

Testing the Solution
^^^^^^^^^^^^^^^^^^^^
Testing or *validating* the solution can be done with a test tool or an actual :abbr:`NAS (Network Access Server)`. 

A simple end-to-end test involves sending a credential set, typically a username and an OTP and getting a response: either an :code:`access-accept` meaning the authentication was *successful* or an :code:`access-reject` meaning (most likely) the password was incorrect and authentication therefore *unsuccessful*.

Test cases
----------

The most basic test plan might look as follows:

+------+-----------------------+-----------------+-------------------------+
| TC # | Test steps            | Expected result | Actual result           |
+======+=======================+=================+=========================+
| TC 1 | Send valid OTP        | access-accept   |                         |
+------+-----------------------+-----------------+-------------------------+
| TC 2 | Send invalid OTP      | access-reject   |                         |
+------+-----------------------+-----------------+-------------------------+
| TC 3 | Disable service       | (no response)   |                         |
+------+-----------------------+-----------------+-------------------------+

.. attention::
   The latest SafeNet FreeRADIUS agent implements a :code:`do_not_respond` policy to allow for customer controlled failover. Earlier versions of the agent may respond with Access-Reject when service is unavailable.

Generally available test tools includes radclient and NTRadPing.

**Radclient** is a RADIUS client program *included* as part of FreeRADIUS and is bundled with the SafeNet FreeRADIUS agent. It can send arbitrary RADIUS packets to the RADIUS server and show the response(s) and has the benefit of being able to facilitate load testing by specifying requests per second etcetera. To learn how to use radclient please refer to the `FreeRADIUS wiki <https://wiki.freeradius.org/config/Radclient>`__ 

Example radclient request (from FreeRADIUS wiki):

::

    echo "User-Name = test" | /usr/local/bin/radclient localhost:1812 auth s3cr3t
    echo "User-Name=test,User-Password=mypass,Framed-Protocol=PPP " | /usr/local/bin/radclient localhost:1812 auth s3cr3t
    echo "Message-Authenticator = 0x00" | /usr/local/bin/radclient localhost:1812 auth s3cr3t


**NTRadPing**, while more limited in capabilities than radclient, does offer the advantage of running on Windows and being more portable and easier to use. For that reason, testing with NTRadPing is described in detail in the following sections. 


Testing with NTRadPing
======================
Installation
------------
To get started with the NTRadPing test tool `download <https://community.microfocus.com/dcvta86296/attachments/dcvta86296/OES_Tips/148/1/ntradping.zip>`__ and unpack the tool and then double-click on :file:`NTRADPING.EXE`.

.. tip::
   Download `NTRadPing 1.5 <https://community.microfocus.com/dcvta86296/attachments/dcvta86296/OES_Tips/148/1/ntradping.zip>`__ here.


Configuration
-------------
Configuring NTRadPing is generally self-explanatory. As such the following table captures the key configuration parameters.
All other parameters can be assumed optional. Note that for your convenience settings are persisted when closing the app.

+-------------------+------------------------------------------------------+
| Property          | Explanation                                          |
+===================+======================================================+
| RADIUS Server     | FreeRADIUS host IP/FQDN                              | 
+-------------------+------------------------------------------------------+
| Port              | 1812                                                 |
+-------------------+------------------------------------------------------+
| RADIUS Secret key | Shared secret (see clients file)                     |
+-------------------+------------------------------------------------------+
| User-Name         | SAS/STA username                                     |
+-------------------+------------------------------------------------------+
| Password          | OTP                                                  |
+-------------------+------------------------------------------------------+
| Request type      | Authentication Request                               |
+-------------------+------------------------------------------------------+

Example requests and responses
------------------------------

.. thumbnail:: /images/freeradius/testingNTRadPingAccessAccept.png
   :title: Figure: Access-Accept is returned on successful authentication.
   :show_caption: true
|

.. thumbnail:: /images/freeradius/testingNTRadPingAccessReject.png
   :title: Figure: Access-Reject is returned on unsuccessful authentication.
   :show_caption: true
|

.. thumbnail:: /images/freeradius/testingNTRadPingNoResponse.png
   :title: Figure: An empty response is returned (configurable) on misconfiguration or service unavailable.
   :show_caption: true
|

.. attention::
   The latest SafeNet FreeRADIUS agent implements a configurable :code:`do_not_respond` policy to allow for customer controlled fail-over. Depending on configuration the agent may respond with Access-Reject when service is unavailable.

How to test using OTP (TOTP/HOTP)
---------------------------------
To test with an authenticator programmed in either :abbr:`TOTP (Time-based One-time Password)` or :abbr:`HOTP (HMAC-based One-time Password)`, e.g. *MobilePASS+* or *OTP 110*:

.. note::
   Refer to **Configuration** section above for base configuration!

#. Generate an OTP using your SafeNet authenticator
#. Enter the OTP into the **Password** field
#. Click :guilabel:`&Send` to submit the request

How to test using Push OTP
--------------------------
To test with an authenticator that supports Push Authentication, e.g. *MobilePASS+*:

.. note::
   Refer to **Configuration** section above for base configuration!

#. Enter any character on the **Password** field
#. Click :guilabel:`&Send` to submit the request
#. Accept the Push notification on your mobile device

How to test using Challenge-Response
------------------------------------
To test with an authenticator programmed in challenge-response mode, e.g. *SMS*:

.. note::
   Refer to **Configuration** section above for base configuration!

#. Enter any character on the **Password** field in order to trigger the :code:`Access-Challenge`

   .. thumbnail:: /images/freeradius/testingNTRadPingSubmitS.png
      :width: 80%   
      :title: Figure: Triggering a challenge.	  
	  
#. In the drop-down in the lower right corner, select **State**
#. In the field next to aforementioned drop-down, carefully enter the state value as shown in the access-challenge
#. Update the **Password** field with the response (typically as received over SMS)

   .. thumbnail:: /images/freeradius/testingNTRadPingChallengeResponse.png
      :width: 80%
      :title: Figure: Sending the response.	  
	  
#. Click :guilabel:`&Send` to submit the request

Load testing
============
Multiple tools exist for load or stress testing a RADIUS server although public availability of such tools is increasingly scarce. GUI based tools supporting Windows includes **Dee's RADIUS Client** and **Evolynx RADIUS Load Test**. 

Both these tools (shown below) can be configured to run a set number of requests per second as well as use multiple threads. Important to note is that said tools works with *fixed* passwords only and as such may not generate the same computation effort on the authentication server as would the use of *One Time* Passwords (OTP's).

To perform more *realistic* load testing One Time Passwords should be used. For Thales internal purposes, in Operations and Engineering, *additional* test tools exist that can generate OTP from the MP-1 software authenticator. This is expected to be equivalent to using MobilePASS+ in terms of server-side computation effort. While MP-1 is deprecated, customers with an existing license capacity may be able to build a test tool leveraging MP-1 SDK and a command line test tool such as **radclient**.

.. warning::
   
   Do not under any circumstances run load/stress testing against any Thales cloud or partner service hosting such as SafeNet Trusted Access. Thales performs testing on behalf of its customers. If you want to load test the FreeRADIUS agent then it must be done in isolation from SafeNet Trusted Access.  

.. thumbnail:: /images/freeradius/deesRadiusClientThreads.png
   :title: Figure: Using Dee's RADIUS Client tool.
   :show_caption: true
|

.. thumbnail:: /images/freeradius/evolynxRadiusLoadTest.png
   :title: Figure: Using the Evolynx RADIUS Load Test tool.
   :show_caption: true
| 

   
   
Upgrading 
^^^^^^^^^
The SafeNet FreeRADIUS configuration script is used for upgrading *any* current installation, regardless of version. That is: the procedure for upgrading is the same, but the actions performed by the process differs according to detected version.

For instance, when upgrading from version :code:`2.x` installed RPMs packages of the FreeRADIUS Agent and FreeRADIUS Updater are removed whereas performing the upgrade on version :code:`3.x` stops, renames and finally removes the existing Docker container image.

.. warning::
   
   Upgrading the SafeNet FreeRADIUS agent does not maintain current configuration. Therefore always make sure you document all settings, including the RADIUS client table prior to performing the upgrade.  

To upgrade the SafeNet FreeRADIUS agent:

#. Open a terminal to the FreeRADIUS agent host
#. Document all current settings and clients
#. Run the :code:`FreeRADIUSv3.sh` by executing the following command

   ::

        sudo sh ./FreeRADIUSv3.sh

#. Follow further instructions found :ref:`here <Running-the-configuration-script>`

