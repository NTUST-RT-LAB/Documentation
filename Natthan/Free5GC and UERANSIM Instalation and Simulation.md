# Free5GC

## Introduction
### Network Topology
![](https://i.imgur.com/b03YWHC.png)

### 5G Network Elements & Function
#### AMF (Access and Mobility Management Function)
The AMF will take rol in knowing the subscriber mobility, registration and security. 
* It will know the location of the subscriber, or at least the default location of the subscriber when it is required
* The AMF will also be communication with other NE for subscriber-related informations
* Take part in the authentication of the subscriber and also providing GUTI which is a temporarty identifier for the UE, after the device send a SUCI key and have been decrypted by the UDM. 

#### SMF (Session Management Function)
The SMF will be taking the role session management related task such as PCF (Policy Control Function) interaction for the data network profile, UPF (User Plane Function) and IP allocation. 
* The SMF will communicate with the PCF for QoS and Policy related information
* Assigning IP to UE for IP required task
* Selection of UPF(Because there are usually multiple upf in the 5G Core)

#### UDM (Unified Data Management)
The UDM will be taking the role in access authorization, registration and mobility management and data network profile. It contains the registry of the subscriber information and data network profile. 
#### NRF (Network Repository Function)
The NRF will be responsible for managing the network functions in the 5G Core for registration and the discovering of new NF

#### UPF (User Plane Function)
The UPF is where all the user data will go through after the PDU session is established. It will also become an anchor point for the GnodeB, because the UE will probably change GnodeB over time but could have the same UPF. Other than that the UPF will also implement the QoS and Policy that is set by the PCF. 

#### NSSF (Network Slice Selection Function)
The NSSF will be responsible to decide which UE will use which slice of the network

#### AUSF (Authentication Service Function)
The AUSF will basically responsible for the authentication of the UE

#### AF (Application Function)
The AF is basically an application and is connected to the PCF for related charging-task. 

#### NEF (Network Exposure Function)
The NEF will be responsible for internet and third party related task. So the NEF will provide an API to those services. 

#### NSSAAF (Network Slice Specific Authorization and Authentication Function)
The NSSAF will be responsible when a specific Network Slice is required to have Authorization and Authentication services. 

#### PCF (Policy Control Function)
The PCF will be responsible for the dynamic policy decision based on the resources in the present time. It will also enforce policy base on the subscriber level of subscription to ensure the level of QoS that the UE will be given. 
## Personal Notes on 5G Core
The target is to make PDU sessions with customized QoS Flow based on the UE user subscription

First the UE will connect to the amf which the AMF will be taking the role of Access point of the 5G Core. In this session building process the UE will send a SUCI key that will be sent to the UDM to be Decrypted. Then the UE will get a GUTI which is a temporary key for the identifier of the UE. After that and the session is built by the SMF where it will applies also policies from UDM and PCF to the UPF. Then the UE will have the session through the GnodeB (Next Generation) UPF and Data Network Sequencially.


### Additional Notes
In case of changing GNode B the UPF will remain the same but the GNode B changes

The smf will also assign upf to the UE. SMF will also responsible for assigning IP address if the traffic requires it. 

Network elements in the 5g core is registered in the NRF in case of registration and deregistration. 

PCF applies dynamic policies on its network condition even if the subscriber have high subscription level 

AMF knows the location, atleast the default location of the UE in the network
## Instalation Setup on VM
### Install Ubuntu on VM
![](https://i.imgur.com/OOdxVgC.png)

### Create Ubuntu VM clone for Free5GC
![](https://i.imgur.com/EIRab1Z.png)

## Installing Free5GC
### Installing control plane elements
![](https://i.imgur.com/MaC8FpU.png)

### Compiling network function services
![](https://i.imgur.com/4e2X94O.png)

### Intalling User Plan Function(UPF)
![](https://i.imgur.com/QwHzeO3.png)

### Installing WebConsole
![](https://i.imgur.com/N0xWxQj.png)

## Testing the Free5GC on VM
Setup for testing
```
cd ~/free5gc
make upf
chmod +x ./test.sh
```

### TestRegistration
```
./test.sh TestRegistration
```

![](https://i.imgur.com/VXDzShp.png)

### TestGUTIRegistration
```
./test.sh TestGUTIRegistration
```

![](https://i.imgur.com/ME6kSYX.png)

### TestServiceRequest
```
./test.sh TestServiceRequest
```

![](https://i.imgur.com/MZ2xvUz.png)

### TestXnHandover
```
./test.sh TestXnHandover
```
![](https://i.imgur.com/Vvh5woO.png)

### TestDeregistration
```
./test.sh TestDeregistration
```
:::success
![](https://i.imgur.com/6UAAQS6.png)
:::


![](https://i.imgur.com/6UAAQS6.png)

### TestPDUSessionReleaseRequest
```
./test.sh TestPDUSessionReleaseRequest
```
![](https://i.imgur.com/JKvJYLp.png)

### TestPaging
```
./test.sh TestPaging
```
![](https://i.imgur.com/5Wh9GFg.png)

### TestN2Handover
```
./test.sh TestN2Handover
```
![](https://i.imgur.com/7L1N380.png)

### TestReSynchronization
```
./test.sh TestReSynchronization
```
![](https://i.imgur.com/mn6vDVo.png)

### TestULCL
```
./test_ulcl.sh TestRequestTwoPDUSessions
```
![](https://i.imgur.com/oKxq8Yz.png)

## Amazon Web Server
### Amazon EC2
Amazon EC2 is a service that is provided by Amazon Web Services that provide scalable computing resources in their cloud service. By using the Amazon EC2 , user will be able to launch and manage virtual servers which is called EC2 Instance. In this opportunity Free5GC will be installed and configured on an EC2 instance



## Install Free5GC in Amazon Web Server

### Prequisite
#### Make EC2 Instance in AWS
![](https://i.imgur.com/a2zNu1j.png)
#### Golang Version
```
Go Version
```
![](https://i.imgur.com/oMh7jBd.png)

#### Control-plane Supporting Packages
```
sudo apt -y update
sudo apt -y install mongodb wget git
sudo systemctl start mongodb
```
![](https://i.imgur.com/a5173iu.png)

![](https://i.imgur.com/yOVYTHu.png)

![](https://i.imgur.com/bkwdjBa.png)

#### User-plane Supporting Packages
```
sudo apt -y update
```
![](https://i.imgur.com/TQKw4vM.png)
```
sudo apt -y install git gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
```
![](https://i.imgur.com/mXZ4RQC.png)

#### Linux Host Network Settings
```
sudo sysctl -w net.ipv4.ip_forward=1
```
![](https://i.imgur.com/qvQwn1V.png)

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
![](https://i.imgur.com/Ng6bnNm.png)

```
sudo iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1400
sudo systemctl stop ufw
sudo sysctl -w net.ipv4.ip_forward=1
```
![](https://i.imgur.com/prxy7K0.png)

#### Install Control Plan Elements
```
cd ~
git clone --recursive -b v3.2.1 -j `nproc` https://github.com/free5gc/free5gc.git
cd free5gc
```
![](https://i.imgur.com/DmMTaJT.png)

![](https://i.imgur.com/iyZM64t.png)

```
cd ~/free5gc
make
```
![](https://i.imgur.com/NIB9bpr.png)

#### Install User Plane Function (UPF)
```
git clone -b v0.6.8 https://github.com/free5gc/gtp5g.git
cd gtp5g
make
sudo make install
```
![](https://i.imgur.com/ca9wY5Y.png)

![](https://i.imgur.com/8rXLf20.png)

![](https://i.imgur.com/DFtrZqv.png)

#### Install WebConsole
```
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get update
sudo apt-get install -y nodejs yarn
```
![](https://i.imgur.com/b8Qla34.png)

![](https://i.imgur.com/YFIy27P.png)

## Testing the Free5GC in AWS
```
cd ~/free5gc
make upf
chmod +x ./test.sh
```
![](https://i.imgur.com/1qXhfCT.png)


```
./test.sh TestRegistration
```
![](https://i.imgur.com/cjby1qy.png)

```
./test.sh TestGUTIRegistration
```
![](https://i.imgur.com/YojPQXQ.png)

```
./test.sh TestServiceRequest
```
![](https://i.imgur.com/jhvp5UK.png)

```
./test.sh TestXnHandover
```
![](https://i.imgur.com/AHsbKaz.png)

```
./test.sh TestDeregistration
```
![](https://i.imgur.com/stlNBmS.png)

```
./test.sh TestPDUSessionReleaseRequest
```
![](https://i.imgur.com/N6DpEuh.png)

```
./test.sh TestPaging
```
![](https://i.imgur.com/3pcr6T7.png)

```
./test.sh TestN2Handover
```
![](https://i.imgur.com/yqTm8vx.png)

```
./test.sh TestNon3GPP
```

```
./test.sh TestReSynchronization
```
![](https://i.imgur.com/QXNn4cT.png)

```
./test_ulcl.sh TestRequestTwoPDUSessions
```
![](https://i.imgur.com/OYngx55.png)

## UE/RAN Simulation Installation
:::info
Reference : https://www.free5gc.org/installations/stage-3-sim-install/
:::
1. Download EURANSIM
```
cd ~
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
git checkout v3.1.0
```
:::info
![](https://i.imgur.com/njIdiFp.png)

![](https://i.imgur.com/XH4lhXi.png)

:::

2. Update and Upgrade UERANSIM VM
```
sudo apt update
sudo apt upgrade
```
:::info
![](https://i.imgur.com/soJGUg5.png)
:::

3. Installing Required Tools
```
sudo apt install make
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic
```

:::spoiler
![](https://i.imgur.com/ReeAzbw.png)
![](https://i.imgur.com/5kuX5Mz.png)
![](https://i.imgur.com/gc1jIue.png)
![](https://i.imgur.com/ITellEs.png)

:::

4. Build UERANSIM
```
cd ~/UERANSIM
make
```
:::info 
![](https://i.imgur.com/JHFlyOT.png)
:::

5. Installing Free5GC Web Console
Removing absolete tools
```
sudo apt remove cmdtest
sudo apt remove yarn
```
:::spoiler
![](https://i.imgur.com/sfzv8Kx.png)
:::

Installing Node.js and Yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install -y nodejs yarn
```
:::spoiler
![](https://i.imgur.com/we521Ap.png)
:::
Build Web Console
```
cd ~/free5gc
make webconsole
```
::: spoiler
![](https://i.imgur.com/c9i2zPc.png)
:::

6. Use Web Console to add an UE
Starting up Web Console
```
cd ~/free5gc/webconsole
go run server.go
```
:::info
![](https://i.imgur.com/SVxThZQ.png)
:::

:::success
![](https://i.imgur.com/nZNjdun.png)
:::
:::spoiler
Username : admin
Password : free5gc
:::
 
Make new subscriber data
:::success
![](https://i.imgur.com/kn9Mww8.png)
:::
:::spoiler
Change the Operator Code Type into OP
:::
7. Configure Free5GC and UERANSIM Parameters
Configuring amfcfg.yaml
```
cd ~/free5gc
nano config/amfcfg.yaml
```
:::info
Change the IP to the Free5GC IP
![](https://i.imgur.com/C4PEB7Y.png)
:::

Configuring smfcfg.yaml
```
nano config/smfcfg.yaml
```

:::info
![](https://i.imgur.com/gijGUsZ.png)
:::

Configuring upfcfg.yaml
```
nano config/upfcfg.yaml
```

::: info
![](https://i.imgur.com/BScAKcU.png)
:::

Configuring GnodeB in UERANSIM
```
nano free5gc-gnb.yaml
```
::: info
![](https://i.imgur.com/gOPyBCQ.png)
:::

Configuring UE in UERANSIM
```
nano free5gc-ue.yaml
```
:::info
![](https://i.imgur.com/7o48jYg.png)
:::

## Testing UERANSIM
1. Running Free5GC
```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
cd ~/free5gc
./run.sh
```
::: success
![](https://i.imgur.com/UPfdmpf.png)
:::

2. Running GnodeB on UERANSIM
```
cd ~/UERANSIM
build/nr-gnb -c config/free5gc-gnb.yaml
```
:::success
![](https://i.imgur.com/G5nwhFK.png)
:::

3. Running UE on UERANSIM
```
cd ~/UERANSIM
sudo build/nr-ue -c config/free5gc-ue.yaml
```
:::success
![](https://i.imgur.com/EZqdzXC.png)
:::

4. Testing Connection
Testing connection with Free5GC
```
ping 192.168.56.101
```
:::success
![](https://i.imgur.com/CJotJEn.png)
:::
  
Checking if the network interface is formed
:::success
![](https://i.imgur.com/1Mt8kRd.png)
:::
  
Now testing connection through the new interface
```
ping -I uesimtun0 google.com
```
:::success
![](https://i.imgur.com/qsl1b44.png)
:::

### Result Log
Results can be accessed [here](https://drive.google.com/drive/folders/1YWd9WY0FElDPYwE0SWhRfZP-hIuexycp?usp=sharing).

## Notes on Development of ORAN Strategy MNO-A Architecture Facing Merger MNO C+D Using Tactical Benchmarking in Java

### Seminar Notes
The current MNO-A now have reach the limit of its performance in terms of frequency capacity. While facing the challenge, MNO C+D merger is outrunning the MNO-A performance. On this issue, the paper have the solution to resolve this by implementing ORAN strategy in the MNO-A Architecture. By implementing ORAN, MNO-A can simplify infrastructure development strategies, including the plan of deploying 2000 sites with 323.3 MHz frequency bandwidth and service coverage operation of 26.89%. In term of cost, the investment could reduce Capex (Capital Expenses) by 36.7% and Opex (Operational Expenses) by 17.36% with business value of IRR more than 100% and BCR (Benefit Cost Ration) that exceeds 6.18 or 37% more than traditional architecture. 






