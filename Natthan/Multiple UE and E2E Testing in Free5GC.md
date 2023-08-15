# Multiple UE and E2E Testing
## Setting up new VM for another UE
**Clone** the previous UERANSIM VM
:::info
![](https://i.imgur.com/c1EBNRq.png)
:::

### Change the VM configuration
Change the hostname of the VM into **UERANSIM2**

```
sudo nano /etc/hostname
```
:::info
![](https://i.imgur.com/A0maiO3.png)
:::
```
sudo nano /etc/hosts
```
:::info
![](https://i.imgur.com/Ukghnpr.png)
:::

Change the static IP address from previous configuration**192.168.56.102** into **192.168.56.103**
```
cd /etc/netplan
sudo nano 00-installer-config.yaml
```
:::info
![](https://i.imgur.com/IXA1ecS.png)
:::

**Apply** the changes in the netplan
```
sudo netplan apply
```
:::success
![](https://i.imgur.com/HmiQ66l.png)
:::

## Configuring the new UERANSIM
### Create a **new** subscriber in the Free5GC Web Console
**Run** the web console
```
cd ~/free5gc/webconsole
go run server.go
```
::: info
![](https://i.imgur.com/DojIY0t.png)
:::

::: warning
Username is **admin** and password **free5gc**
:::


Create a new subscriber and change the SUPI into **208930000000004** and the Operator Code Type into **OP**
::: info
![](https://i.imgur.com/j9OR99i.png)
:::

Now there will be 2 subscribers registered
::: success
![](https://i.imgur.com/MRL8oKC.png)
:::

### Changing the UERANSIM configuration file
We have to change 2 file configuration **free5gc-gnb.yaml** and **free5gc-ue.yaml**

### Configuring the GnodeB
In this simulation, the testing will be done through **2 Gnode B and 2 UE**. For this reason the GnodeB IP from **192.168.56.102** into **192.168.56.103**
```
sudo nano ~/UERANSIM/config/free5gc-gnb.yaml
```
:::info
![](https://i.imgur.com/vdQv7Gc.png)
:::

### Configuring the UE
Make sure that the configuration for the UE is **the same** as it is in the web console

```
sudo nano ~/UERANSIM/config/free5gc-ue.yaml
```
::: info
![](https://i.imgur.com/EXdj2JY.png)
:::

## Testing E2E connection
First, run the **free5GC**
```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
cd ~/free5gc
./run.sh
```
::: spoiler
1. The First command is for enabling IP forwarding, which allows traffic to be forwarded from one interface to another
2. The second command is to configure the NAT using ip tables utiity that add a rule to the NAT table to masquerades/change the source ip address with the interface specified which is ens33
3. The third command is to stop UFW (Unwanted Firewall) service that may cause problem in sending packets
4. The fourth command is to add FORWARD chain rule that enable all packets to be forwarded
:::
:::success
![](https://i.imgur.com/Xr3OtlT.png)
:::

Run the the 2 **GnodeB**
```
cd ~/UERANSIM
build/nr-gnb -c config/free5gc-gnb.yaml
```
First GnodeB,
:::success
![](https://i.imgur.com/NSgs6Uu.png)
:::

Then the second
:::success
![](https://i.imgur.com/gjbZ63a.png)
:::


Now run the 2 **UE** 
```
cd ~/UERANSIM
sudo build/nr-ue -c config/free5gc-ue.yaml
```
First UE,
:::success
![](https://i.imgur.com/BAByzLD.png)
:::

Then the second one,
:::success
![](https://i.imgur.com/Wl0X7yA.png)
:::

Now it can be seen that the first UE have the ip **10.60.0.1** in the **uesimtun0** interface.
:::info
![](https://i.imgur.com/dE6fY4G.png)
:::
The second one have the IP 10.60.0.2 in the uesimtun0 interface
:::info
![](https://i.imgur.com/o8yyxBE.png)
:::

Now we are going to test the connection by sending packet from the first UE through the uesimtun0 interface vice versa. 

**First UE**
:::success
![](https://i.imgur.com/y1HUWD7.png)
:::

**Second UE**
:::success
![](https://i.imgur.com/AJJdIPp.png)
:::

## Simulation Log
The log to these simulation can be accessed [here]()