# 3GPP TS 23.041

3GPP is a partnership project from a number of Standards Development Organizations (SDOs) from around the globe initially to develop technical specifications and protocol  for mobile communication

In this note, we will spesifically talk about the 3GPP TS (Technical Spesification) 23.041, which is about **the Cell Broadcast Short Message Service (CBS) for GSM (Global System for Mobile Communication) and UMTS (Universal Mobile Telecommunication System)** and also **Public Warning System (PWS) for GSM, UMTS, E-UTRAN, and NG-RAN**

## 1. General Description
The CBS Service permits a number of unacknowledged general CBS message to be broadcast to all receivers within a particular region and broadcast to defined geographical area known as cell broadcast area. CBS messages may originate from a number of Cell Broadcast Entities (CBEs), which are connected to the Cell Broadcast Centre. CBS messages are then sent from the CBC to the cells, in accordance with the CBS's coverage requirements.

A CBS page comprises of 82 octets, or equals to 93 characters when using default character set. Up to 15 of these pages may be concatenated to form a CBS message. Each page of this message will have the same message identifier and serial number. Using this information, the MS/UE can identify to ignore re-broadcasts of already received messages

CBS messages are broadcast cyclically by the cell at a frequency and for a duration agreed with the information provider. The frequency at which CBS messages are repeatedly transmitted will be dependent on the information that they contain; for example, dynamic information such as road traffic information, will require more frequent transmission than weather information.

To permit mobiles to selectively display only those CBS messages required by the MS/UE user, CBS messages are assigned a message class which categorises the type of information that they contain and the language (Data Coding Scheme) in which the CBS message has been compiled, so that the user is able to ignore message types that he does not wish to receive

PWS provides a service that allows the network to distribute warning messages on behalf of public authority. PWS enables the distribution of ETWS, CMAS (aka WEA), KPAS and EU-Alert warning messages in GSM, UMTS, E-UTRAN, and NG RAN.

## 2. Network Architecture
The network architecture differ for GSM, UMTS, EPS, and 5GS

### 2.1 GSM Network Architecture
![](https://i.imgur.com/q0lGnWn.png)

Figure 2.1 The basic network structure of CBS in the GSM

### 2.2 UMTS Network Architecture
![](https://i.imgur.com/P3XhHBF.png)

Figure 2.2 The basic network structure of CBS in the UMTS

### 2.3 E-UTRAN Network Architecture
![](https://i.imgur.com/zWxCZQH.png)

Figure 2.3 The basic network structure of PWS in the E-UTRAN

### 2.4 5GS Network Architecture
![](https://i.imgur.com/hHsPzdK.png)

Figure 2.4.1 5GS PWS system architecture, using service-based interfaces between CBCF and AMF

![](https://i.imgur.com/ckdJLST.png)

Figure 2.4.2 5GS PWS architecture in reference point representation without PWS-IWF

![](https://i.imgur.com/fTiJHOi.png)

Figure 2.4.3 5GS PWS architecture in reference point representation with PWS-IWF

## 3. CBE Functionality
CBE Functionality is outside of the scope of this 3GPP spesifications, but it is assumed that the CBE is responsible for all aspects of formatting CBS messages, including the splitting of a CBS message into a number of pages.

## 4. CBS Functionality
In 3GPP the CBC is integrated as a node in the core network.

The CBC may be connected to several BSCs/RNCs/MMEs/PWS-IWFs. The CBC may be connected to several CBEs. The CBC shall be responsible for the management of CBS messages including:
* Allocation of serial numbers;
* Modifying or deleting CBS messages held by the BSC/RNC/eNodeB/NG-RAN node;
* Initiating broadcast by sending fixed length CBS messages to a BSC/RNC/eNodeB/NG-RAN node for each language provided by the cell
* Determining the set of cells to which a CBS message should be broadcast, and indicating within the Serial Number the geographical scope of each CBS message;
* Determining the time at which a CBS message should commence being broadcast;
* Determining the time at which a CBS message should cease being broadcast and subsequently instructing each BSC/RNC/eNodeB/NG-RAN node to cease broadcast of the CBS message;
* Determining the period at which broadcast of the CBS message should be repeated;
* Determining the cell broadcast channel in GSM, on which the CBS message should be broadcast.
* When CBS transmits emergency messages, allocation of "emergency indication" to differentiate it from normal CBS messages, including the "Cell ID/Service Area ID list", "warning type", "warning message". If "warning type" is of 'test', only UEs which are specially designed for testing purposes may display warning message.

## 4A. CBCF Functionality
In 3GPP the CBCF is a network function in the 5G core network 
The CBCF may be connected to several AMFs. The CBCF may be connected to several CBEs. The CBCF shall be responsible for the management of CBS messages that is almost the same like the CBS responsibility described above, except that the node is NG-RAN (New Generation Radio Access Network) node 

## 5. BSC/RNC/MME/AMF Functionality
* The BSC/RNC shall interface to only one CBC. A BSC may interface to several BTSs. An RNC may interface to several Node Bs.
* The MME may interface to one CBC or multiple CBCs . An MME may interface to several eNodeBs.
* The AMF may interface to one CBCF or multiple CBCFs. An AMF may interface to several NG-RAN nodes.

### 5.1 BCS/RNC Responsiblity
The BCS/RNC shall be responsible for:
* Interpretation of commands from the CBC.
* Storage of messages from the CBC.
* Scheduling of CBS messages on the CBCH.
* Providing an indication to the CBC when the desired repetition period cannot be achieved.
* Providing to the CBC acknowledgement of successful execution/forwarding of commands received from the CBC.
* Reporting to the CBC failure when a command received from the CBC is not understood or cannot be executed.
* Routing of CBS messages to the appropriate BTSs/Node Bs.
* Generating Schedule Messages, indicating the intended schedule of transmissions 

### 5.2 MME/AMF Responsibility
The MME/ANF shall be responsible for:
* Interpretation of commands from the CBC.
* Providing to the CBC acknowledgement of successful execution/forwarding of commands received from the CBC.
* Reporting to the CBC failure when a command received from the CBC is not understood or cannot be executed.
* Report the Broadcast Completed Area List, the Broadcast Cancelled Area List, PWS Restart Indication and the PWS Failure Indication received from eNB(s)/NG-RAN to all CBCs/CBCFs and PWS-IWFs that the MME/AMF interfaces with.
* Routing of warning messages to the appropriate eNodeBs/NG-RAN in the indicated Tracking Area.
* Sending the Write-Replace Warning Request message to the appropriate eNodeBs/NG-RAN upon receiving warning message transmission request from CBC/CBCF or PWF-IWF.

## 6. BTS Functionality
The BTS is responsible for conveying CBS information received via SMS BROADCAST REQUEST or SMS BROADCAST COMMAND messages over the radio path to the MS.
:::info
This is only for GSM
:::

## 7. MS/UE Functionality
### 7.1 General MS/UE Functionality
The MS/UE has several functionality:
* Have the ability to discard CBS information which is not in a suitable data coding scheme.
* Have the ability to discard a CBS message which has a message identifier indicating that it is of subject matter which is not of interest to the MS/UE.
* Have the ability to detect duplicate messages
* Have the ability to transfer a CBS message to an external device, when supported
* Enable the user to activate/deactivate CBS through MMI
* Enable the user to maintain a "search list" and receive CBS messages with a Message Identifier in the list while discarding CBS messages with a Message Identifier not in the list.
* Allow the user to enter the Message Identifier via MMI only for the 1 000 lowest codes.
* If the emergency indication includes the value for "test", mobile terminals which are not used for testing purpose silently discard the paging message and do not receive the corresponding CBS/warning message.


#### 7.1.1 MS Functionality
MS (Mobile Station) functionality that is differ than UE (User Equipment) functionality:
* Discard sequences transferred via the radio path which do not consist of consecutive blocks
* Optionally skip reception of the remaining block of a CBS message which does not contain cell broadcast information
* Optionally read the extended channel.
* Be capable of receiving CBS messages consisting of up to 15 pages 

#### 7.1.2 UE Functionality
UE (User Equipment) functionality that is differ than MS (Mobile Station) functionality:
* Discard corrupt CBS messages received on the radio interface.
* Be capable of receiving CBS messages consisting of up to 1230 octets in UTRAN or warning messages of up to 9600 octets in E-UTRAN, or NG-RAN.

### 7.2 Duplication Detection Function
The MS/UE uses a common duplication detection function for all messages received in GSM, UMTS, E-UTRAN and NG-RAN.
In a PLMN, upon reception of a new message, the MS/UE shall perform duplication detection on the messages. Those messages that are received from the same PLMN in the certain time period specified by the duplication detection time are subject to duplication detection. The MS/UE shall not perform duplication detection on messages whose duplication detection time has elapsed.

:::info
The value of the duplication detection time to be used by the MS/UE shall be derived from the MCC of the current PLMN as follows:
-	If MCC = 440 or MCC = 441 (Japan), duplication detection time shall be 1 hour;
-	For all other MCCs, duplication detection time shall be 24 hours.
:::

The MS/UE shall check:
1)	Whether the Serial Number associated with the Message Identifier of the new message matches the Serial Number of any of those messages with the same Message Identifier that have been received and displayed to the subscriber 

Additionally, the MS/UE may check:

2)	other criteria for detecting duplicates. An example of such a criterion is whether the actual contents of the two messages is the same.
If criterion 1 is fulfilled and any implemented additional checks are also met, then the MS/UE shall consider the new message as duplicated and shall ignore it.

For ETWS, duplicate message detection shall be performed independently for primary and secondary notifications.

### 7.3 ePWS Functionality
The ePWS functionality consists of the ePWS language-independent content functionality and the ePWS disaster characteristics functionality as follows:
1)	UEs with user interface which support the ePWS language-independent content functionality and which are capable of displaying text-based warning messages should be capable of displaying the language-independent content mapped to an event or a disaster (e.g. character such as Unicode based pictogram mapping to a disaster) that is part of user information contained in the content of a warning message transparently passed from CBC to UEs.
2)	UEs with user interface which support the ePWS language-independent content functionality and which are incapable of displaying text-based warning messages should be capable of mapping message identifiers of received warning messages to language-independent contents stored in those UEs. When a warning message is received, such a UE should be capable of displaying of a language-independent content stored in the UE mapped from the message identifier of the received warning message.
3)	UEs with no user interface which support the ePWS disaster characteristics functionality should be capable of identifying the characteristics of a disaster derived from the message identifier of a received warning message in order to take appropriate action.


## 8. E-UTRAN and NG-RAN CBC Protocol
In telecommunication, protocol is a set of rules and format that allows two or more entities of a communications system to exchange information. Protocol is needed to create consistency and universality for the sending and receiving of messages

In this part, the main topic is about the protocol of CBC in E-UTRAN and CBCF in NG-RAN

### 8.1 E-UTRAN Protocol
![](https://i.imgur.com/u671OK3.png)

The above picture is the connectivity of CBC in E-UTRAN:
* S1-AP (S1 Application protocol) is the application layer between the eNodeB and MME (Mobile Management Entity)
* The connectivity between eNodeB and MME needs S1-MME, a control plane interface for eNodeB to connect to MME. S1-MME also needs SCTP to ensure that the data sent completely
* SBc-AP (SBc Application protocol) is the application layer between the CBC and MME
* The connectivity between eNodeB and MME needs SBc, an interface between the CBC and MME to be able to transfer warning message

### 8.2 NG-RAN Protocol
![](https://i.imgur.com/5eeYE2V.png)

The above picture is the connectivity of CBCF in NG-RAN:
* NG-AP (NG Application protocol) is the application layer between the NG-RAN and AMF (Access and Mobility Management Function)
* The connectivity between NG-RAN and AMF needs N2, a control plane interface for NG-RAN to connect to AMF. N2 also needs SCTP to ensure that the data sent completely
* HTTP/2 is the application layer protocol for service based interface between the CBCF and AMF
* The connectivity between AMF and CBCF needs Namf, a service based interface between the CBCF and AMF
* NG-AP-CB (NG Application Protocol Cell Broacast):

Besides this protocol, NG-RAN can also connect with CBC with the use of PWS-IWF

### 8.2A NG-RAN Protocol with PWS-IWF
![](https://i.imgur.com/T84HbSG.png)

The above picture is the connectivity of CBC in NG-RAN with PWS-IWF:
* The connectivity of CBC in NG-RAN is almost the same with the connectivity of CBCF in NG-RAN, but the CBCF is replaced with PWS-IWF (Public Warning System - Inter Working Function)
* PWS-IWF is a logical interface to translate N50 to SBc
* The connectivity between AMF and PWS-IWF use N50, service based interface to connect between the AMF and the PWS-IWF
* The connectivity between CBC and PWS-IWF use SBc

:::info
PWS-IWF is used when using NG-RAN but CBC is used instead of CBCF
:::

## 9. Warning Message in E-UTRAN and NG-RAN
### 9.1 Warning Message Delivery
The process of warning message delivery is similar with the process of Cell Broadcast Service to transfer CBS message related to public warning. The process of delivering message is done by broadcasting warning message to MS/UEs within a particular area. Below is the process of delivering warning message in E-UTRAN and NG-RAN.

### 9.1.1 Warning Message Delivery in E-UTRAN
![](https://i.imgur.com/pjYbnos.png)

The above picture is the process of warning message delivery in E-UTRAN:

0. Registration procedures: The process of UE registration, when UE is attached to a network
1. Emergency Broadcast Request: CBE sends emergency information, such as warning message, to the CBC and the information will be authenticated by the CBC
2. Write-Replace Warning Request: After the message is authenticated, CBC will determine which MMEs need to be sent the information, and CBC will send a request to broadcast the information to the specific area to MME
3. Write-Replace Warning Confirm: The MME send back write-replace warning confirm to indicate that the warning message has started the broadcast to the eNodeBs
4. Emergency Broadcast Response: After the CBC received the confirm, the CBC also send back the emergency broadcast response to the CBE to indicate that it has started to broadcast the warning message
5. Write-Replace Warning Request: The MME broadcast the warning message to eNodeBs that matches the area in the warning information
6. Cell Broadcast Delivery: The eNodeBs broadcast the warning message to the UEs
7. User Alerting: The warning message will appear on the UEs if the UEs has been configured to accept warning message
8. Write-Replace Warning Indication: The MME will send back the write-replace warning indication to the CBC that contain the broadcast completed area list the MME has received from eNodeBs
9. The MME determines the success or failure of the warning message delivery based on the write-replace warning response

### 9.1.2 Warning Message Delivery in NG-RAN
![](https://i.imgur.com/PeBslm6.png)


The above picture is the process of warning message delivery in E-UTRAN:

0. Registration procedures: The process of UE registration, when UE is attached to a network
1. Emergency Broadcast Request: CBE sends emergency information, such as warning message, to the CBCF and the information will be authenticated by the CBCF
2. Write-Replace Warning Request: After the message is authenticated, CBCF will determine which AMFs need to be sent the information, and CBCF will send a request to broadcast the information to the specific area to AMF
3. Write-Replace Warning Confirm: The AMF send back write-replace warning confirm to indicate that the warning message has started the broadcast to the NG-RANs
4. Emergency Broadcast Response: After the CBCF received the confirm, the CBCF also send back the emergency broadcast response to the CBE to indicate that it has started to broadcast the warning message
5. Write-Replace Warning Request: The AMF broadcast the warning message to NG-RANs that matches the area in the warning information
6. Cell Broadcast Delivery: The NG-RANs broadcast the warning message to the UEs
7. User Alerting: The warning message will appear on the UEs if the UEs has been configured to accept warning message
8. Write-Replace Warning Indication: The AMF will send back the write-replace warning indication to the CBC that contain the broadcast completed area list the AMF has received from eNodeBs
9. The AMF determines the success or failure of the warning message delivery based on the write-replace warning response

### 9A Warning Message Delivery using 5G Service Based Interface

The implementation of the warning message delivery in 5G is using Service Based Interface. The CBCF use service offered by AMF via the Namf/N50 service based interface. There are a few Namf services that are used by CBCF:
1.  Namf_Communication_NonUeN2MessageTransfer

    NonUeN2Message Transfer is used to initiate or stop broadcast in one or more cells. The AMF shall accept the request and respond to the Network Function Service Consumer immediately. NonUeN2MessageTransfer service operation:
    * PWS Write-Replace-Warning Request message or the Stop-Warning Request message are transferred in an N2 Message Container via the NonUeN2MessageTranfer request operation 
    * Write-Replace-Warning Confirm message or the Stop-Warning Confirm message are returned to the sender (CBCF) via the NonUeN2MessageTranfer response operation
    * The List of TAIs information shall be used by the AMF to determine to which NG-RAN nodes the N2 Message Container needs to be forwarded to.
    * Each NonUeN2MessageTransfer message is uniquely identified by the Message Identifier, the Serial Number IE and the Message Type

2. Namf_Communication_NonUeN2InfoSubscribe

    NonUeN2InfoSubscribe is used to subscribe to the delivery of non-UE specific PWS information from the NG-RAN node, sent via N2 to the AMF. NonUeN2InfoSubscribe service operation:
    * The NF Service Consumer ID is an identifier which is configured in the CBCF or PWS-IWF
    * The N2 information types are WarningIndications or RestartFailure
    * If the N2 information type is WarningIndications then the NF Service Consumer subscribes to receive Write-Replace-Warning Indication messages, and Stop-Warning Indication messages from the AMF
    * If the N2 information type is RestartFailure then the NF Service Consumer subscribes to receive Restart Indications and Failure Indications from the NG-RAN node.

3. Namf_Communication_NonUeN2InfoUnsubscribe

    NonUeN2InfoUnubscribe is used to unsubscribe to stop notifying non-UE specific N2 information from the NG-RAN node
    
4. Namf_Communication_NonUeN2InfoNotify

    NonUeN2InfoNotify is used by AMF to notify a particular PWS event towards the CBCF that has subscribed for the specific information. The AMF receives messages for such PWS events from NG-RAN via N2
    
The message flow for these services operation:

![](https://hackmd.io/_uploads/B1yp3RV83.png)

1. If the CBCF supports reception of Write-Replace-Warning Notifications and Stop-Warning Notifications then the CBCF uses the Namf_Communication_NonUeInfoSubscribe service operation to subscribe to these notifications.
2. The CBCF sends a Write-Replace-Warning Request message or a Stop-Warning-Request message to the AMF using the Namf_Communication_NonUeN2MessageTransfer service operation. The AMF returns a Namf_Communication_NonUeN2MessageTransfer response message.
3.	The AMF determines from the List of TAIs IE to the NG-RAN nodes the N2 Message Container shall be forwarded to.
4.	The AMF forwards the messages included in the N2 Message Container to the selected NG-RAN nodes via N2 and receives a response from the NG-RAN nodes.
5.	The AMF may aggregate the responses it has received from the NG-RAN nodes.
6.	The AMF forwards the response(s) as Write-Replace-Warning Notification(s) or Stop-Warning Notifications to the CBCF using the Namf_Communication_NonUeNotify service operation if the CBCF has subscribed to receiving these notifications in step 1.


