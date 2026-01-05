# FYC - NEXTCLOUD

## Global architecture

![architechture.png](.attachments.390/architechture.png)

## Install and configure Opensense

For this example, we are gonna use opnsense as a firewall + reverse proxy ! It's open source, free ti use and super easy to understant when you are not familliar with network configurations

### Opnsense proxmox config

First, download the ISO and upload it to your Proxmox, then create a new VM just like so 👍

![{5217B519-20F9-4111-A52A-34FC283BE25E}.png](.attachments.390/%7B5217B519-20F9-4111-A52A-34FC283BE25E%7D.png)

Name it as you want

![{DBB453BB-340C-4E5B-8FE0-70E6519931CD}.png](.attachments.390/%7BDBB453BB-340C-4E5B-8FE0-70E6519931CD%7D.png)

Add your ISO file of the OPNSense and Linux type kernel 6.x the ISO file can be found here :  
<https://opnsense.org/download/>

 Use DVD image to have a .iso then upload it onto your proxmox server ! 

![{9B764843-00D0-485F-A532-DD0114AF824D}.png](.attachments.390/%7B9B764843-00D0-485F-A532-DD0114AF824D%7D.png)

Let SeaBIOS, by default and Default display and i440fx machine type let the default SCSI controller and add Qemu Agent  
- i440fx represent the type of virtual machine but both options works here, we just left default !  
- BIOS SeaBIOS : Default simple bios here, but does not support secure boot but we don’t need it. If you want to enable secure boot, choose the other option UEFI Bios and enable TPM.But mostly not needed on firewalls like this and maybe FreeBSD does’’t support it at all so stay with SeaBIOS as it is easier and the default option.

![{C1A33AD3-8A5D-41EB-9087-BEA42D75B204}.png](.attachments.390/%7BC1A33AD3-8A5D-41EB-9087-BEA42D75B204%7D.png)

Add a bit of storage 60 Go should be enough let the SCSI controller by default and no cache

![{173223C5-ADAB-4471-8805-8A9564EF74BC}.png](.attachments.390/%7B173223C5-ADAB-4471-8805-8A9564EF74BC%7D.png)

Opnsesne is not very demanding in resources so 2 CPU should be enough and it can run with only one if needed but be careful, all your traffic will goes through this one so we need some performances to get a high internet speed ! Let the x86-64-v2-AES by default

![{EA2F7101-12C1-4403-AB9E-D22AE302B9F4}.png](.attachments.390/%7BEA2F7101-12C1-4403-AB9E-D22AE302B9F4%7D.png)

Now add 4Go of RAM, this is a lots but it's needed for the installation (+3Go) but we can lower it afterwards if needed !

![{9E8F6BA0-8A9D-4220-B5BF-C5DB14801C20}.png](.attachments.390/%7B9E8F6BA0-8A9D-4220-B5BF-C5DB14801C20%7D.png)

Add the network, only one interface for now, we will add a second later just before starting our OPNsense ! Let the defaults, VirtIO controller and vmbr0 default bridged interface  
VirtIO is the default and the most efficient driver here on virtualization for Linux kernels. With windows, if we want to use this driver, we have to install it first cause it is not native. 

![{53AAF833-4DBE-412A-A6B4-AC78604BDA8A}.png](.attachments.390/%7B53AAF833-4DBE-412A-A6B4-AC78604BDA8A%7D.png)

Now that's done, uncheck the "start after created" checkbox we need to add a second network interface first

![{AB6B3A32-EBC2-4488-815B-7676780F950B}.png](.attachments.390/%7BAB6B3A32-EBC2-4488-815B-7676780F950B%7D.png)

Now go in the new VM section, and add a new network interface, same vmbr0 bridged default interface !

## Opnsense installation

  
We are good to go, now you can start the VM and process the opnsense installation 👍  
Once the VM is started, you should see a login screen, default credentials for OPNSense are : "root" and "opnsesne", but we are gonna install it first, because it's not done yet ! By default, it boots on a temporary tryal mode, so we have to install it to make it permanent.

login with the following credentials to trigger the installation process :  
login : "installer" pass:  "opnsense"

Follow the instructions to install OPNSense, no redundancy so Ex-fat, don't forget to change the default root password before restarting your VM !  
  
Of course if you are setting this up for a company, it is always recommended to use redoundancy in case the firewall breaks or dies. You’ll have saves and recover so you don’t have to reconfigure it after a loss. You cna even push the thing further and have 2 instances of OPNSense with config synchronized to make some load balancing if you have a big infrastructure and you need a lot of speed.

Once it's done, you should have a fresh install for your OPNSense, a default WAN interface should have been created, get your ipv4 address and try <https://opnsense-ip>, you should see a login screen ! Login with your root credentials

![{120D8D23-F5EB-472C-B4B5-85ECF755843C}.png](.attachments.390/%7B120D8D23-F5EB-472C-B4B5-85ECF755843C%7D.png)

![{0222124D-5684-4335-A53B-215A07585224}.png](.attachments.390/%7B0222124D-5684-4335-A53B-215A07585224%7D.png)

Now for security reason, at the moment you add a new LAN network in OPNSense, the default connection to the web interface will shutdown, because you are accessing it from your WAN address which is not recommended ! So be careful, thoses steps are crucial if you don't want to lose access to your OPNSense web interface !

## Create VLANs and LAN interface

First thing to do is to create a VLAN for our nextcloud service ! Well basically, this is a web service accessible publicly, so this service should go in a DMZ, so we are gonna create a DMZ VLAN for our nextcloud VM :  
In Interfaces/Devices/VLAN, add this :

![{A00E9B44-9317-4BAD-861E-0BE93AAF22D9}.png](.attachments.390/%7BA00E9B44-9317-4BAD-861E-0BE93AAF22D9%7D.png)

![{B14D46F4-1BCF-40EA-A575-334126702262}.png](.attachments.390/%7BB14D46F4-1BCF-40EA-A575-334126702262%7D.png)

For me, the WAN took vtnet0 so I'll take the vtnet1 to transport my LAN and VLANs !

- The vtnet1 is the second network interface which is not connected already. Our LANs networks will be configured with this interface.
- The vtnet0 is the WAN network interface where internet comes from. 

Once this done, we need to assign a new interface :

![{F65E9541-EDE7-4BEF-8046-6E047CB2CC35}.png](.attachments.390/%7BF65E9541-EDE7-4BEF-8046-6E047CB2CC35%7D.png)

in Interfaces/Assignments, look for your VLAN Device and add it as a new interface !  
Then we will configure the interface to create a new VLAN 10.0.3.1/24 with no DHCP server :

![{6908324E-C04E-4043-817D-E782E99E90F2}.png](.attachments.390/%7B6908324E-C04E-4043-817D-E782E99E90F2%7D.png)

![{64B5135F-8803-4ABC-943A-AD3A458E10D3}.png](.attachments.390/%7B64B5135F-8803-4ABC-943A-AD3A458E10D3%7D.png)

Now with this done, we have a new VLAN 10.0.3.1/24

- Why no DHCP ? :  
  For security reasons ! Here this network we have just created will act as our DMZ → the zone where traffic will come from the internet ! So it is an untrusted zone ! We do not allow devices to acquire an IP address automatically cause this could be dangerous. We ‘ll have to control each device we add on this network and make sure no one is being added without us notifying !
- IPV6 ? :   
  In most cases IPV6 is not required inside a local network ! It may allow us to have more IP on our sub network but we don’t need so much addresses ! Maybe on specific cases it will be activated and configured but here there is no need too and IPV4 still much more understandable and easier than IPV6.

## Create Nextcloud Ubuntu VM

I choose to use Ubuntu 24.04 LTS as it's a simple distro all build, with NVIDIA graphic drivers compatibility. It is also recommended by NextCloud to install it on this distro because it's the most documented and used for NextCloud !

But we have some extra config to make :

![{B3858806-B826-4327-9FD2-B32B9F7C36F0}.png](.attachments.1187/%7BB3858806-B826-4327-9FD2-B32B9F7C36F0%7D.png)

![{B3858806-B826-4327-9FD2-B32B9F7C36F0}.png](.attachments.390/%7BB3858806-B826-4327-9FD2-B32B9F7C36F0%7D.png)

Choose the Ubuntu ISO you imported on your proxmox, download it and upload it before if not already done !

link to Ubuntu downloads : <https://ubuntu.com/download/server#manual-install-tab>

![{2F1A29E5-E1D7-4648-A602-B3154F825028}.png](.attachments.390/%7B2F1A29E5-E1D7-4648-A602-B3154F825028%7D.png)

Now this part changes a bit, select standard VGA for the display method and choose q35 type machine with OVMF UEFI boot part, add Qemu agent and select your desired EFI storage, you can also pre-enroll keys !

- Why q35 :   
  It is the recommended type of machine to support GPU pass through and that’s what we are gonna do here, to have a more fluid desktop experience we can even make some AI later for our NextCloud using the GPU !
- Why OVMF UEFI ? :   
  Same here, we have a Ubuntu Server, wich supports UEFI and secure boot, we can enable it. It also helps for the graphic driver compatibility ! The Bios of the Graphic card is also UEFI so it will only be compatible with UEFI boot systems.

![{2C0D5261-622A-4FAA-A09A-DBCE7105B578}.png](.attachments.390/%7B2C0D5261-622A-4FAA-A09A-DBCE7105B578%7D.png)

Now be shure to add extra space on this one because it is gonna be your cloud, so basically where all your data will be stored ! Along with the AI models which are not so lightweight ! As long as i have a SSD, i've checked the box SSD emulation, but this depends on your configuration !

![{21063803-D3D5-4F11-8450-FFAD579825E6} (2).png](.attachments.390/%7B21063803-D3D5-4F11-8450-FFAD579825E6%7D%20%282%29.png)

Now for the CPU part, i'm gonna be generous, because we'll have clam AV (antivirus for files) running in background with Talk, FullTextSearch which takes 1G RAM and 1 VCPU, if we want to work with AI, it is also recommended to add extra cores if you can !

![{4F0879F2-55E6-433E-861B-F57B4203F9EF}.png](.attachments.390/%7B4F0879F2-55E6-433E-861B-F57B4203F9EF%7D.png)

Same for the RAM, here i put 16 Go because some models doesn't fit in my GPU memory, so it will take space within my RAM ! And we want a smooth nextcloud experience ! But depending on what you want to do with your nextcloud you can adjust theses settings. For a familly usage 5-6 pers max you can have only 4-5 CPU and 6-8 Go RAM.

![{AB76BA97-0BC3-49FA-8DB4-20D9BB8FCB18}.png](.attachments.390/%7BAB76BA97-0BC3-49FA-8DB4-20D9BB8FCB18%7D.png)

For the Network here, don't forget to select vmbr0 (the same interface than the LAN on OPNSense) and add the VLAN 30 with no firewall (we run OPNSense on our own). This will tag all the packets going out from the interface with VLAN 30, so OPNSense will detect this and manage the traffic coming from this VM, but we need to configure it first !

![{4A5048B5-6D5E-44B6-BF53-7C4655BEF247}.png](.attachments.390/%7B4A5048B5-6D5E-44B6-BF53-7C4655BEF247%7D.png)

Now with this done, you can create and power on your VM !

## NextCloud Ubuntu configuration

In this part we are going trough all the installation of the NextCloud Ubuntu VM !

Install your Ubuntu by following the interactive install, if you need to configure the network because you have taken a network install, just put the ip 10.0.3.2, gateway 10.0.3.1 and /24 ! Your Ubuntu should automatically connect and have access to internet ! If you have internet access issue, you can also configure DNS with 8.8.8.8 or 8.8.4.4 temporally !

once you are logged into your fresh Ubuntu install first update and upgrade it :

```
sudo apt update && sudo apt upgrade -y
```

Then shutdown your VM and add the PCIE GPU device : 

![{336E6691-3D54-4B8C-8206-44CE673BC364}.png](.attachments.390/%7B336E6691-3D54-4B8C-8206-44CE673BC364%7D.png)

![{86525BA8-51FD-411E-B3EB-1A8C9E1EB2A6}.png](.attachments.390/%7B86525BA8-51FD-411E-B3EB-1A8C9E1EB2A6%7D.png)

Most of the time, you will have too PCIE devices detected for your graphics card :   
- Video  
- Audio   
Select the one finishing with 00.0 as it is most of the time the video part.

Restart the machine,

then install the NVIDIA driver with the Ubuntu Additional driver tools :

![{1D553C05-969C-4546-93D0-03A0A0D32D73}.png](.attachments.390/%7B1D553C05-969C-4546-93D0-03A0A0D32D73%7D.png)

![{ED30C32C-4CD8-4D81-8719-C34252D1ECA7}.png](.attachments.390/%7BED30C32C-4CD8-4D81-8719-C34252D1ECA7%7D.png)

Once the driver installed, reboot one last time and you should be done and you should see this in the NVIDIA X server settings app :

![{AC3859FC-171B-46CB-BBB3-3315D5D54BA1}.png](.attachments.390/%7BAC3859FC-171B-46CB-BBB3-3315D5D54BA1%7D.png)

## Install docker

Now I'll just follow the docker documentation and install it on my system :

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# sudo usermod -aG docker tstraub
```

Don't forget to logout and re-login into your account to apply the group changes ! Try docker with :

```
docker ps
```

👍 Docker is now installed on our system !

## Setting up nextcloud your domain name

We are almost done setting up our NextCloud, we now need to install NextCloud via docker AIO which is what they recommend, because it is all in one, and simple to configure ! But ! We are gonna use HAProxy managed from our OPNSense instance, so we need to get rid of the default reverse proxy which is a Caddy by default !  
In their documentation, they specify this URL for a proxy-less install using your own reverse proxy :  
<https://github.com/nextcloud/all-in-one/blob/main/reverse-proxy.md>

According to the documentation the correct command for us to launch NextCloud wil be :

```
sudo docker run \
--init \
--sig-proxy=false \
--name nextcloud-aio-mastercontainer \
--restart always \
--publish 8080:8080 \
--env APACHE_PORT=11000 \
--env APACHE_IP_BINDING=0.0.0.0 \
--env APACHE_ADDITIONAL_NETWORK="" \
--env SKIP_DOMAIN_VALIDATION=false \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
ghcr.io/nextcloud-releases/all-in-one:latest
```

- \--init : initialize NextCloud for the first time with domain name setup and containers setup
- \--sig-proxy=false : to specify that we don’t want the integrated proxy as we will run our own proxy on OPNSense (HAProxy)
- \--name : to specify the name for the container, i’ve let the default here cause it is already named correctly
- \--restart always : Will restart the container if a fatal error occur, to try fixing things. It will also allow NextCloud Containers to start  on system startup ! Great if you have to plan a maintenance and shutdown the Ubuntu server !
- \--publish 8080:8080 : This is the port to publish for the first install process (and only for the install process). you can change is as you want, but don’t forget to connect the the right port to configure your NextCloud afterwards 
- \--env APACHE_PORT 11000 : This passes an environment variable to the Apache container that will expose our NextCloud. This parameter is important as we will specify this port to our HAProxy to connect to our NextCloud.
- \--env APACHE_IP_BINDING=0.0.0.0 : This is the range of addresses on which our Apache is gonna listen for connections ! 0.0.0.0 means all addresses will be allowed to request and use our NextCloud. Here we could have set 10.0.3.1 which is the address of our firewall and HAProxy. This would have only allowed the HAProxy and firewall to access our NextCloud which is a good practice cause all the connections to our NextCloud will first go through our firewall ! But in order not to complicate things i’ll let 0.0.0.0 here. So if i want to change my sub-net i don’t have to restart my NextCloud with the modified address. Plus We are already behind a firewall so this is extra security we can let on the side. 
- \--env APACHE_ADDITIONAL_NETWORK : Same as the previous parameter, if you want to add 2 different precise sub-nets !
- \--env SKIP_DOMAIN_VALIDATION=false : Set to true, this will bypass the domain validation and you’ll have access to an insecure HTTP NextCloud. This really useful when you want to test it or if you don’t have a domain name already or if you just wan to use it locally at your house with no public access ! We will let it to false here as we want to open it publicly and add a domain name to use it !
- \--volume : Thoses parameters sets the default volumes for the NextCloud config persistence, let the default parameters to avoid conflicts and errors ! The volume “docker.sock:ro” specify a socket connection in read-only (:ro) This will allow NextCloud to deploy new applications and integrations via our docker really easily. This is called ExApps (ExternalApps) it is also needed for AI local integration ! But be careful as we are opening our docker socket, this implies security questions ! Be careful of what applications you are installing, i recommend to only use verified apps or to make a code review yourself before installing anything !

You can also add an extra

```
--env EXTERNAL_STORAGE="/mnt/usb" \
--volume /mnt/usb:/mnt/usb
```

If you want to connect a local storage like a USB mounted volume into your NextCloud ! You can configure access to it afterwards with the external storage plugin !  
'“/mnt/usb” should be your local Ubuntu server mounted device ! If you have an external HDD you would like to use on NextCloud, you’ll have to pass it first to the proxmox VM config by adding USB raw device. Then you’ll have to mount it on Ubuntu with the mount command ! Be sure you can access your files before specifying this extra param !

enter this command and your NextCloud server will start at <https://localhost:8080> for the initial setup configuration !

First you'll have to **save your secret passphrase**, don't forget to write it down !

Then NextCloud will ask you for your domain name in order to connect to it and use your own certificate and DNS, but we first need to configure a new Domain name for our NextCloud and we need to configure the reverse proxy in OPNSense !

## Configure domain and HTTPS

I'll be using IONOS to buy and manage my domain name server, but you can use whatever domain name provider you like !

I've bought a new domain name tsmaster.space to host our NextCloud :

![{7BCE3878-383A-4485-9582-5F09E5F72E6F}.png](.attachments.390/%7B7BCE3878-383A-4485-9582-5F09E5F72E6F%7D.png)

now I'm gonna configure it to point to my public so let's change the DNS records for AA and AAAA wich is the default web configuration for a DNS :

![{7C6B2FEB-FC63-4D3C-B42F-7F94802C913E}.png](.attachments.390/%7B7C6B2FEB-FC63-4D3C-B42F-7F94802C913E%7D.png)

Perfect now let's manage our certificate :

![{F63BF1EF-0B80-4C7D-BC78-21E5DB42E031}.png](.attachments.390/%7BF63BF1EF-0B80-4C7D-BC78-21E5DB42E031%7D.png)

![{166F5113-0C68-4B29-9AA0-E101B877B640}.png](.attachments.390/%7B166F5113-0C68-4B29-9AA0-E101B877B640%7D.png)

We can see here that the validation uses the DNS to register itself so we are gonna use this in OPNSense + HAProxy

### DNS validation how it works ?

Here is a little schema of how DNS validation for a domain name usage with HTTPS is working :

![Letsencrypt-DNS-authentication-1510102574.png](.attachments.1187/Letsencrypt-DNS-authentication-1510102574.png)

  
Benefits :

- Automatic Domaine Name validation
- Automatic certificate renewing every 60 days no manual actions needed

### ACME configuration

First, check if you don't have any update for your OPNSense :

![{C5D7A2D0-CA94-4AE9-A2D0-DB2BBA9CE5CF}.png](.attachments.390/%7BC5D7A2D0-CA94-4AE9-A2D0-DB2BBA9CE5CF%7D.png)

If you have some, update it and restart if needed, always keep your firewall update when exposing on the internet !

the go to the "Plugin" section

System > Firmware > Plugins

Check community plugins on the right side.

![{69912DDC-2CFA-4C33-B595-094873CAC207}.png](.attachments.390/%7B69912DDC-2CFA-4C33-B595-094873CAC207%7D.png)

Install the HAProxy plugin and the ACME client plugin, we are gonna use ACME to make the domain check validation ! Enable the plugin in it’s settings and create a ACME account with a valid email and register it here :

![{F1AB28FA-ED10-4E47-BD34-CED86B7C5101}.png](.attachments.390/%7BF1AB28FA-ED10-4E47-BD34-CED86B7C5101%7D.png)

First we need to activate the ACME Client Plugin and to enable HAProxy integration // add capture

Then add a new challenge type DNS :  
For IONOS, i got a key from the API management <https://developer.hosting.ionos.com/>

You can add a key here and use the API key in OPNSense

![{B41D1DF7-D6D5-4103-936E-DB1044ACA9EB}.png](.attachments.390/%7BB41D1DF7-D6D5-4103-936E-DB1044ACA9EB%7D.png)

Last step is to add a new certificate in the certificates section like so :  
Just use the ACME account and Challenge type you've just created

![{726C4EA9-AF34-4EA4-A4D1-AB0EEC67BC29}.png](.attachments.390/%7B726C4EA9-AF34-4EA4-A4D1-AB0EEC67BC29%7D.png)

Then click on ISSUE/RENEW all certificates to start the validation process ! This will go and check with your API key that your domain is configured and it will validate the certificate via DNS.

Once we've done this, you should see your certificate appearing :

![{77AA5A2C-6152-4091-843F-5E82B54752FE}.png](.attachments.390/%7B77AA5A2C-6152-4091-843F-5E82B54752FE%7D.png)

It should be validated too !

### HAProxy configuration

//little reminder reverse proxy, how it works, add schema

First we need to enable HAProxy :

![{56B216B2-1C4C-4F72-BD8C-F64B84B0AD27}.png](.attachments.390/%7B56B216B2-1C4C-4F72-BD8C-F64B84B0AD27%7D.png)

Then add a real server into the real servers section , the real server is the physical server on which the request is gonna be redirected ! You can totally add multiple servers for the same service to make a load balancing web service !

![{EC796ACF-D263-4D29-85D8-B605DA29C28E}.png](.attachments.390/%7BEC796ACF-D263-4D29-85D8-B605DA29C28E%7D.png)

Add 2 servers :

10.0.3.2 for your NextCloud which is our VM

127.0.0.1 for the ACME validation challenge which is our OPNsense (automatic and should appear already)

![{C71A37D6-B969-41AD-BB3A-C7FBE95F08CA}.png](.attachments.390/%7BC71A37D6-B969-41AD-BB3A-C7FBE95F08CA%7D.png)

The default NextCloud AIO docker uses the port 11000 to connect HTTPS as we can see in the docker ps command 👍

```
tstraub@tstraub-ubuntu-master:~$ dps #(docker ps)
nextcloud-aio-apache	5dabfd3b6d17	Up 21 hours (healthy)	80/tcp, 0.0.0.0:11000->11000/tcp
...
```

Now we need to create some back-end pools, add one for NextCloud and one for ACME is created automatically. Set the Layer to HTTP and set the real server to the one created above for your NextCloud

![{F8521BF7-81C0-4A68-8F47-38FEE2DEF1A4}.png](.attachments.390/%7BF8521BF7-81C0-4A68-8F47-38FEE2DEF1A4%7D.png)

![{97C35A3B-63C9-4CBC-ADD6-DEED8E7D93FF}.png](.attachments.390/%7B97C35A3B-63C9-4CBC-ADD6-DEED8E7D93FF%7D.png)

![{CE788286-11EC-4F5E-B1FB-6E86E012EE61}.png](.attachments.390/%7BCE788286-11EC-4F5E-B1FB-6E86E012EE61%7D.png)

Add 2 conditions :  
1 for the nextcloud matches host tsmaster.space

1 for the acme client (automatic)

![{65DBAAD6-685F-49BD-A40E-7E3362D6581E}.png](.attachments.390/%7B65DBAAD6-685F-49BD-A40E-7E3362D6581E%7D.png)

Finally add some rules :

![{AB9EDB29-8F0B-454E-A800-F50493E48D5E}.png](.attachments.390/%7BAB9EDB29-8F0B-454E-A800-F50493E48D5E%7D.png)

Finally add a public front :

Set a listening address for the front, in order to get the http and https connexions, we are gonna user thoses ports on the public WAN interface of our opnsense ! so 192.168.1.170:443 We could also create a redirection rule from http to https for this nextcloud server ! (We'll do it later)

![{690D56B6-3703-4F08-B32E-D172AF19FA40}.png](.attachments.390/%7B690D56B6-3703-4F08-B32E-D172AF19FA40%7D.png)

You'll have to create the rules first before adding them !

![{56FB1AF9-A06E-465E-B130-E4EF307D335C}.png](.attachments.390/%7B56FB1AF9-A06E-465E-B130-E4EF307D335C%7D.png)

![{4982A716-B3C6-4A66-8B4D-5D7ED1D8174E}.png](.attachments.390/%7B4982A716-B3C6-4A66-8B4D-5D7ED1D8174E%7D.png)

![{91CCEA70-07BF-49F9-8CD4-67582C8E9B27}.png](.attachments.390/%7B91CCEA70-07BF-49F9-8CD4-67582C8E9B27%7D.png)

Now last part, we have to redirect the 443 requests comming into our Box to our opnsense so let's add a redirect rule into our Box :

First change the default web access ports for your Box :

![{244BC677-0D8E-4D69-8981-82B207DE9B8F}.png](.attachments.390/%7B244BC677-0D8E-4D69-8981-82B207DE9B8F%7D.png)

Then create a redirect rule for 443 and 80 ports, redirect to Opnsense !

![{A5C5869A-1B22-43C8-9ABF-4D6CC4E308CD}.png](.attachments.390/%7BA5C5869A-1B22-43C8-9ABF-4D6CC4E308CD%7D.png)

You'll have to change the opnsense default web ui port too ! It can not be 443 anymore since we are gonna use it for our reverse proxy !

In System/Settings/Administration change the web port to 8443 too !

![{079F4B32-4AA4-4834-9709-B424A2B0C556}.png](.attachments.390/%7B079F4B32-4AA4-4834-9709-B424A2B0C556%7D.png)

Now we are good to go !

## Configure nextcloud

Now we are finished with the https certificates validations and DNS register, now we can go back to our nextcloud install and we can reenter the domaine tsmaster.space and we can submit the domain with the blue button ! If everything is configured correctly, you should have a green success message and access to the container managment screen !

https://localhost:8080/containers

On this new page, you can configure the containers to add , download and run with your nextcloud instance ! You can comehere and change that every time you need !

Select the containers you want to install :

![{906E5855-6B0B-4630-AC77-8BECA9913B5E}.png](.attachments.390/%7B906E5855-6B0B-4630-AC77-8BECA9913B5E%7D.png)

I recommand thoses ones + somes other usefull community containers :

![{5E18D977-EC54-4E5C-BD38-1DE66544DEF2}.png](.attachments.390/%7B5E18D977-EC54-4E5C-BD38-1DE66544DEF2%7D.png)

Once you've selected your containers to install and start, don't forget to validate and deploy them. Once this step is done, you should see an invation link to your new nextcloud !

But ! We don't have any credentials by default so we are gonna change the default password for the admin user :

On your terminal on the Ubuntu Nexcloud VM :

```
docker exec -it nextcloud-aio-nextcloud bash 
# exec bash inside nextcloud container
php occ user:resetpassword admin
```

Then type the new password 2 times ! You can now connect into your nextcloud with the default admin account !

## Recap

So now we have :  
Acces to the Box : change to 8443 port (https://192.168.1.254:8443

Access to the opnsense : change to 8443 port (<https://10.0.3.1:8443>)

Access to our nextcloud at <https://tsmaster.space>