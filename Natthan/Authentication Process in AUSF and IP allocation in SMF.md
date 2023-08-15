# Authentication Process in AUSF and IP allocation in SMF

## General Registration Process

![](https://i.imgur.com/kCLH1m0.png)

## Registration Process
[Reference](https://www.etsi.org/deliver/etsi_ts/123500_123599/123501/16.06.00_60/ts_123501v160600p.pdf) 

There are 2 possibilities for registration in the 5G System, which is RM-Registered state and RM-Deregistered state. RM-Registered state is when the UE is registered to the network, while in RM-Deregistered otherwise. 
### RM-Deregistered
- attempt to register with the selected PLMN using the Initial Registration procedure if it needs to receive service that requires registration
- remain in RM-DEREGISTERED state if receiving a Registration Reject upon Initial Registration . enter RM-REGISTERED state upon receiving a Registration Accept.

When the UE RM state in the AMF is RM-DEREGISTERED, the AMF shall:
- when applicable, accept the Initial Registration of a UE by sending a Registration Accept to this UE and enter RM-REGISTERED state for the UE ; or
- when applicable, reject the Initial Registration of a UE by sending a Registration Reject to this UE .

### RM-Registered
* perform Mobility Registration Update procedure if the current TAI of the serving cell  is not in the list of TAIs that the UE has received from the network in order to maintain the registration and enable the AMF to page the UE; 
* perform Periodic Registration Update procedure triggered by expiration of the periodic update timer to notify the network that the UE is still active. 
* perform a Mobility Registration Update procedure to update its capability information or to re-negotiate protocol parameters with the network; 
* perform Deregistration procedure , and enter RM-DEREGISTERED state, when the UE needs to be no longer registered with the PLMN. The UE may decide to deregister from the  network at any time. 
* enter RM-DEREGISTERED state when receiving a Registration Reject message or a Deregistration message.
* The actions of the UE depend upon the 'cause value' in the Registration Reject or Deregistration message.

When the UE RM state in the AMF is RM-REGISTERED, the AMF shall:

* perform Deregistration procedure , and enter RM- DEREGISTERED state for the UE, when the UE needs to be no longer registered with the PLMN. The network may decide to deregister the UE at any time;
* perform Implicit Deregistration at any time after the Implicit Deregistration timer expires. The AMF shall enter
* RM-DEREGISTERED state for the UE after Implicit Deregistration;
* when applicable, accept or reject Registration Requests or Service Requests from the UE.

## UE IP Address Management
The UE IP address management includes allocation, renewal and release of the UE IP address. 

 If the UE has a pre-defined rule or configuration which is a UE Local Configuration that contain a PDU Session Type of either IPv4, IPv6 and IPv4v6 that matches the requested PDU Session Type, it will use that rule or configuration. But if there is no pre-defined rule or configuration available, it will not include that specific PDU Session Type in the request message.
 
  if there is no matching URSP rule and no matching UE Local Configuration, the UE shall set the
requested PDU Session Type during the PDU Session Establishment procedure. A UE which supports IPv6 and IPv4 shall set the requested PDU Session Type "IPv4v6". A UE which supports only IPv4 shall request for PDU Session Type "IPv4", etc. 

The SMF selects PDU Session Type of the PDU Session as follows:
- If the SMF receives a request with PDU Session Type set to "IPv4v6", the SMF selects either PDU Session Type "IPv4" or "IPv6" or "IPv4v6" based on DNN configuration, subscription data and operator policies.
- If the SMF receives a request for PDU Session Type "IPv4" or "IPv6" and the requested IP version is supported by the DNN the SMF selects the requested PDU Session type.

## SMF and UPF interaction (PFCP Protocol)
[Reference](https://youtu.be/saMtTHO2GFk)
### Background
So there are the evolution of architecture from the 4G to the 5G. Common gateway used in the previous architecture is the SGW and the PGW. Then comes along the CUPS architecture that seperate the control plane and the userplane of the architecture. Now in the 5G architecture there will be the SMF and the UPF that seperates control and user plan entirely. The protocol for communication between the control plane (SMF) and the user plane is the PFCP (Packet Forwarding Control Plane)

![](https://i.imgur.com/LFCH5Mj.png)



There are severals way the CP (Control Plane) will control the UP (User Plane). There are
* Establishing the PFCP session
* Modifying the PFCP Session 
* Deleting the PFCP session
The CP will control it by applying rules to the UP, such as the PDR(Packet Detection Rule), FAR(Forward Action Rule), QER(QOS Enforcement Rules), UAR(Usage Report Rule), and BAR(Buffering Action Rule)

Overall the packet will be send through the UPF where there will be PFCP session lookup to find the matching PDR and apply the instruction contain with it such as FAR,QER and URR

##### PFCP Session Establishment
First of the AMF will send the nsmf-pdusession/sm-context to the SMF, then the SMF will send the PFCP Establishment req. In this step the SMF will send the PDR to the UPF which contain FAR, QER and URR. 

The PDR for the uplink traffic will set the source interface into access(From the User), meanwhile for downlink traffic the source interface will be core (From the internet)

#### FAR
Now for the uplink traffic the Forward parameter will be set to true and the buffer into false, so that the packet from the user will be sent through the UPF and then the internet

As for the downlink traffic the Buffer parameter will be set to true, meanwhile the forward into false. This will make the data from the internet to be temporaryly stored in the buffer before sending it to the user

#### QER
In the PDR that is sent by SMF to the UPF there will also be QER which include QFI (QOS Flow information) and MBR(Maximum Bit Rate). This information basically inform the UPF regarding the QoS Policy such as the upload and download rate. 

### UAR
In the UAR that is sent to the UPF will contain the quota/volume for the user. Thats why the volume parameter will be set into true and the Volume threshold will be set. 

After the request is sent, the UPF will response the PFCP Establishment, giving the smf the TEID and the IP address specific to the current session. 

### PFCP Modification
When the session is established, there will be changes due to the state of the user, data sent from the internet sitting in the buffer, etc. Here are the cases PFCP modification is done

#### N3 Tunnel Establishment
In this case the AMF will sent nsmf-pdusession/sm context/modify to the SMF and the SMF will send PFCP session modification req, where the forward will be set into true and the destination will be set to access. This means the data that is sitting in the buffer will be sent/forwarded to the User. 

#### Volume Threshold
The usage of the user will be sent to the SMF from the UPF. The report will contain on how much does the user have use their quota, both in uplink and downlink. 

Now if more quota is needed then the SMF will send nchf-converged charging data/ update to the CHF, which will give the update back in response to the SMF. After this the SMF will update the rules in the UPF by sending the PFCP session modification request. 

#### Session Deactivation
In the case when the user is going idle, the amf will send a nsmf-pdusession/sm context/modify request for deactivation to the SMF. After this the SMF will send a FAR update to the UPF where the buffer will be set into true and the destication interface to access. This means that this session will receive data from the internet and have it hold in the buffer.

#### Downlink Data Report
In this case if the data from the internet is sitting in the buffer, the UPF will send a PFCP session report req to the SMF by sending the downlink data report and the paging policy so that the SMF will send it to the AMF and finally to the User to change it from its current idle mode. 

#### Session Activation
When the User get notified, that there is a packet waiting for them the user will send a request from the UE to the AMF to the SMF a nsmf-pdusession/sm context/modify/activated request. Then the SMF will send a PFCP session modification req which will update the buffer parameter into false and the forward into true. This will allow the packet sitting in the buffer to be sent to the user. 

#### VoNR call
In this case the PCF will send a policy wo the SMF, which will update the rules in the UPF. The SMF will send information such as the network instance etc. 

#### Session Retrieve
This case happens when a user is changing from the 5G network to the 4G network. In this case the AMF will send a request to the SMF that will ask the s5/s8-U and IP/TEID information. Then the SMF will send a request that will update the network instance into s5s8realm and in response the UPF will send the TEID and the IP. The SMF then will forward it back to the user. 

### PFCP Deletion
In the case that the PFCP is going to be deleted. The SMF will recieve a request for session deletation. Then the SMF will send a PFCP session deletion req to delete all the rules that have been applied in the UPF. Then the UPF will give the response that the rules have been deleted and the remaining quota will be sent through the SMF to the CHF. Then the session will be deleted




