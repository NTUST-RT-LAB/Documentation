# Installing UERANSIM - a UE/RAN Simulator - Joshevan
UERANSIM is a simulator for user equipment and Radio Access Network, or simply a simulator for a 5G mobile phone and a base station. It can be used to test 5G core network and also to study the 5G system
## 1. Install ueransim VM
In this part, we will create the UERANSIM VM by creating a new VM, make sure that it can access internet and can be accessed using SSH, and also setting the IP Address to 192.168.56.102
* Create a new VM name ueransim
![](https://i.imgur.com/FbQ8BlO.png)

* Make sure the VM has an internet connection
```
ping 8.8.8.8
```
![](https://i.imgur.com/rGu1cKA.png)
:::info
The command is used to test to ping google IP Address to ensure that the VM has an internet connection
:::

* Make the VM accessible via SSH
```
sudo apt install openssh-server
```
![](https://i.imgur.com/UnfjAbO.png)
:::info
The given commands is used to install open SSH server to make the VM accessible via ssh
:::
* Make the Host-only network interface have static IP address 192.168.56.102
    1. Go to the setting of the VM, Network then enable network adapter 2 then set it into host-only adapter
    ![](https://i.imgur.com/MAHjZAA.png)
    
    2. Enter the setting inside the VM, Network then choose setting of the interface that is host-only network (in this case, enp0s8 interface) 
    ![](https://i.imgur.com/JqMKixN.png)
    
    3. Click the IPv4 tab, then choose IPv4 Method manual, then input the IP Address manually (192.168.56.102), after that reboot the VM
    ![](https://i.imgur.com/gJAsI79.png)
   
 

* Test SSH to ueransim VM from guest OS (Windows)
```
ssh ueransim@192.168.56.102
```
![](https://i.imgur.com/dMx4giZ.png)
    
* Try to ping 192.168.56.102 from free5gc
![](https://i.imgur.com/L1wSBiz.png)

* Try to ping 192.168.56.101 from ueransim
![](https://i.imgur.com/zinMK1A.png)

## 2. Install UERANSIM
In this part, we will install UERANSIM by downloading all the required tools, download the source code then build the UERANSIM
* Update and upgrade ueransim VM first:
```
sudo apt update
sudo apt upgrade
```

![](https://i.imgur.com/4RusjKK.png)

![](https://i.imgur.com/b12ykt8.png)
:::info
These commands is used to update and upgrade the VM
:::

* Install Required Tools
```
sudo apt install make
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```

![](https://i.imgur.com/M5pOOyn.png)

![](https://i.imgur.com/JMSXeu9.png)

![](https://i.imgur.com/R4jRDrv.png)

![](https://i.imgur.com/wDOUwKu.png)

![](https://i.imgur.com/ToIsqql.png)

:::info
These commands is used to install required tools
:::


* Download UERANSIM
```
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
git checkout v3.1.0
```

![](https://i.imgur.com/pgPysGh.png)

:::info
These commands is used to download UERANSIM from remote repository (Github)
:::

* Build UERANSIM
```
make
```
![](https://i.imgur.com/SdbGqrY.png)

![](https://i.imgur.com/UxCiPhP.png)

## 3. Install free5GC Web Console
Free5GC Web Console is a Graphical User Interface (GUI) to connect to the free5gc database from Web Interface, so it is easier to interact with the database for tasks like adding, updating, deleting, displaying subscribers data from the free5gc

:::spoiler Modify Subscribers data without using Web Console

:::warning

If we do not use web console, we have to interact with the database (MongoDB) manually to modifiy subscribers data. It can be done by entering the MongoDB shell and then use query manually.
Below are the steps to modify subscribers data manually through the MongoDB shell:
* Enter the mongodb shell
```
mongo
```
![](https://i.imgur.com/Rqg8XRC.png)

* Connect to free5gc database
```
use free5gc
```
![](https://i.imgur.com/L8GqsaF.png)

* Show all collections
```
show collections
```
![](https://i.imgur.com/x8Fbmuk.png)

* If we want to show all the data in the collection, we can use
```
db.collection().find().pretty()
```
* For example
```
db.subscriptionData.provisionedData.amData.find().pretty()
```
![](https://i.imgur.com/HCu0T0o.png)

The command will return the output in JSON format. We can also filter the displayed output by giving argument to the find method




* If we want to update data in the collection, we can use
```
db.collection.updateOne()
```

* If we want to delete data in the collection, we can use
```
db.collection.deleteOne()
```

:::


* SSH into free5GC
```
ssh free5gc@192.168.56.101
```
![](https://i.imgur.com/vGWICgo.png)

* Build webconsole
```
cd ~/free5gc
make webconsole
```
![](https://i.imgur.com/UTkYP42.png)
:::info
These commands is used to build Web Console in free5gc

:::

## 4. Use WebConsole to Add an UE
In this part, we will run the web console and add an UE using the web console
* Start Web Console Server
```
cd ~/free5gc/webconsole
go run server.go
```
![](https://i.imgur.com/s9RkuwP.png)

* Open URL from host machine (192.168.56.101:5000)

![](https://i.imgur.com/qw1khHf.png)

* Login using username "admin" and password "free5gc"

![](https://i.imgur.com/LKnqMXr.png)

* Choose subscribers and create a new data

![](https://i.imgur.com/SPYUhbY.png)

* Change the Operator Code Type to OP, leave other field unchanged and click submit

![](https://i.imgur.com/fBzJXQ7.png)

* New subscriber

![](https://i.imgur.com/27w9RfO.png)



## 5. Install ueransim VM   
Below is the 5gc architecture:

![](https://i.imgur.com/c3UNL38.jpg)

As shown in the picture, there are few network functions:
* AMF (Access and Mobility Management Function) responsible for authenticating and authorizing user access to the network, managing user mobility within the network, and also working with SMF to manage user session
* UPF (User Plane Function) responsible for forwarding data packets between the RAN and the Core Network, and managing different types of traffic
* SMF (Session Management Function) responsible for forwarding user data from network nodes to data plane processing, and managing session for data and voice services 

From the 5gc architecture and those explanations, we know that those three network functions are related with the connectivity between the UE, RAN, and core network, so have to update the configuration of the free5gc of those three network functions, AMF, SMF, and UPF to change the IP Address of the core network to 192.168.56.1, which is the IP Address of the free5gc
* Edit ~/free5gc/config/amfcfg.yaml
```
cd ~/free5gc
nano config/amfcfg.yaml
```
![](https://i.imgur.com/cP4NgkR.png)

* Replace ngapIpList IP from 127.0.0.1 to 192.168.56.101:

![](https://i.imgur.com/3bwGKcN.png)

* Edit ~/free5gc/config/smfcfg.yaml
```
nano config/smfcfg.yaml
```

![](https://i.imgur.com/RkbAdae.png)

* Edit the entry inside userplane_information / up_nodes / UPF / interfaces / endpoints, change the IP from 127.0.0.8 to 192.168.56.101

![](https://i.imgur.com/p9OriBQ.png)

* Edit ~/free5gc/config/upfcfg.yaml
```
nano config/upfcfg.yaml
```
![](https://i.imgur.com/KAz3ioW.png)

* Change gtpu IP from 127.0.0.8 into 192.168.56.101

![](https://i.imgur.com/qcGd3iu.png)

## 6. Setting UERANSIM
In this part, we have to update the configuration of the ueransim, which is the gnb to change the IP Address of the core network to 192.168.56.1, which is the IP Address of free5gc, and to change the IP Address of the UE to 192.168.56.2, which is the IP Address of the ueransim. We also have to make sure that the UE data is consistent with the one that we made in the free5gc web console.
* SSH into UERANSIM (192.168.56.102)
```
ssh ueransim@192.168.56.102
```
![](https://i.imgur.com/3vnxtFr.png)

* Edit the file ~/UERANSIM/config/free5gc-gnb.yaml

```
nano ~/UERANSIM/config/free5gc-gnb.yaml
```
![](https://i.imgur.com/WblJieu.png)


* Change the ngapIp IP and gtpIp IP, from 127.0.0.1 to 192.168.56.102，and also change the IP in amfConfigs into 192.168.56.101
![](https://i.imgur.com/ek1reME.png)

* Examine the file ~/UERANSIM/config/free5gc-ue.yaml，and see if the settings is consistent with those in free5GC (via WebConsole), for example,

```
nano ~/UERANSIM/config/free5gc-ue.yaml
```
![](https://i.imgur.com/gICuR65.png)

![](https://i.imgur.com/0ua3sS3.png)

The data appear to be the same as what we set in WebConsole.

## 7. Testing UERANSIM against free5GC
In this part, we will test the UERANSIM against the free5gc by running all the component, which is free5gc as core network, gnb as base station, and ue as user equipment. After that, we will try to ping google.com from the created interface to make sure that all the components run properly.
* Run free5gc after making proper changes to the free5GC configuration files

```
cd ~/free5gc
./run.sh
```
![](https://i.imgur.com/P0Wcs0o.png)

![](https://i.imgur.com/YQLihB2.png)

* In the first terminal, execute from ueransim nr-gnb
```
cd ~/UERANSIM
build/nr-gnb -c config/free5gc-gnb.yaml
```
![](https://i.imgur.com/Z2RGIgX.png)

* In the second terminal, execute from ueransim nr-ue with admin right
```
cd ~/UERANSIM
sudo build/nr-ue -c config/free5gc-ue.yaml
```
![](https://i.imgur.com/3HO6p6J.png)

* In the third terminal, ping 192.168.56.101 to see free5gc is alive. Then, use ifconfig to see if the tunnel uesimtun0 has been created (by nr-ue): 

```
ping 192.168.56.101
```
![](https://i.imgur.com/9CQ7yI9.png)

```
ifconfig
```
![](https://i.imgur.com/bGnDTem.png)

* Test Ping
```
ping -I uesimtun0 google.com
```
![](https://i.imgur.com/ra2w8Gt.png)

:::success
The ping gets replies which means free5gC is running properly
:::
