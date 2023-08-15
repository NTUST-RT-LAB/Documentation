# 5G Registration Procedure

[Reference](https://www.eventhelix.com/5g/standalone-access-registration/)

In the 5G registration process there will be several phases that will be included, here are those phases and the explanation regarding what happened in those phase


## Precondition
In this phase, the UE is in idle state and is registered in the Old AMF


## 5G-NR RRC Connection Setup
Here are the sequence of 5G-NR RRC Connection Setup
1. The UE will send a random preamble to the gNodeB with the Zadoff-Chu sequence which is basically a complex sequence so that the signal will be efficiently detected and synchronized
::: spoiler
The preamble is referenced to the RAPID(Random Access Preamble Id), which will send an identifier to each UE and help the base station to distinguish multiple UE that is attempting to initiate connection
:::
2. After sending the preamble, the T300 will initiate in the UE
:::spoiler
T300 is a timer that is applied in 5G network. If in those duration the UE doesn't get any response, the UE will retransmit the preamble
:::
3. The UE will start to decode the PDCCH for the RA-RNTI
:::spoiler
what happened in this phase is basically is by decoding the PDCCH (Physical Downlink Control Channel) for the RA-RNTI(Random Access Radio Network Temporary Identifier), allows the UE to extract relevant control information for communicating with the gNodeB
:::

4. Allocating the temprorary C-RNTI by the gNodeB
:::spoiler
The temporary C-RNTI or  Cell Radio Network Temporary Identifier is an identifier for the UE within the coverage of the gNodeB. It will enable the gNodeB to identify and differentiate multiple UE within their cell
:::

5. Next the RA-RNTI will scrambled a DCI message that provide the UE information regarding the frequency that will be used and the time resources allocated for the transmission. Plus there will be the transport block that will be sent containing the RAR (Random Access Response)

6. Next when the UE detect the DCI message that is sent with the DCI 1_0 format, which contains the transport block containing the RAR, the UE will get the C-RNTI that has been allocated

7. Next the UE will choose a random identity in the domain of a number between 0 and 2 to the power of 38

8. Next the Uplink allocation which also included in the RAR that is recieved previously, the UE will transmit a Msg3 to the gNodeB to establish RRC connection

9. Next the RRC setup request will be sent from the UE to the Gnode B along with the random ue identity

10. The C-RNTI then will scramble the DCI message containing the frequency and time resources that allows the UE to utilize the communication

11. The signaling radio 1 is configured in the GnodeB which allows the control plane messages to be signaled between the UE and the gNodeB

12. The RRC Setup message is sent containing the information required to setup SRB1 and the master cell

13. When the UE receive the RRC setup message the T300 will be stopped

14. The GNodeB will send uplink resource to the UE so the UE can send the RRC Setup Complete Message

15. Finally the UE will send the RRC Setup Complete message with "Registration Request" in the dedicatedNAS-Message Field. 

16. The GnodeB will select the AMF for the session
17. The gNodeB will allocate the RAN UE NGAP ID which will function as an address to the UE on the gNodeB for the AMF 
18. Then the NB sends the Initial UE Message to the selected AMF carrying the "Registration Request" message that was received from the UE, "RAN UE NGAP ID" and the "RRC Establishment Cause".


## Obtaining  the UE Context from the Old AMF
1. The new AMF will request the old AMF the 5G-GUTI of the UE to do context transfer. 
2. The Old UE will then check the registration request, to make sure security and possible malacious attacks. 
3. The old AMF send the UE context to the new AMF
4. The new AMF save the UE context
5. The new AMF will request the UE identity or the SUCI
6. The UE respond to the Identity Request
7. Then the AMF will selec an AUSF for aunthenticating the SUCI

## NAS Authentication and Security
1. The AMF will request the UE authentication vectors and algorithm information from the AUSF. 
2. The AUSF will request the authentication vectors from the UDM
3. The UDM generate the authentication vector
4. UDM gives the authentication vector to the AUSF
5. The AUSF will send the master key which is used by AMF to derive NAS security keys and other security key(s). The SUPI will also be sent to the AMF
6. THE AMF will challend the UE authentication procedure  by sending the key selector, RAND and AUTN to the UE.
:::spoiler
The RAND is a random generated number by the AuC. AUTN is the authentication token which contain the RAND
:::
7. The UE give respond to the Authentication Challenge
8. The AMF will give the security algorithm and related information to the UE and also request the IMEISV from the UE.
:::spoiler
IMEISV are International Mobile Equipment Identity and Software Version is a unique identifier for the UE
:::
9. The UE will send the response containing the IMEISV
10. The new AMF will send a message to the old AMF that the UE is now registered

## Confirming the UE is not blacklisted
1. The UE will obtain the PEI and send it to the 5G-EIR
::: spoiler
The PEI (Public Equipment Identity) can be derived from the device IMEI or other equipment specific Id. The 5G-EIR (5G Equipment Identity Register) stores the equipment identities in a 5G network. 
:::
2. The 5G-EIR will check id the PEI is blacklisted. Then it will report the result to the AMF

## Register with the UDM and obtain the subscription data
1. The new AMF will request registration with the UDM
2. The UDM will send a response code "204 No Content" if sucessful
3. The AMF request the Access and Mobility subscription data
4. UDM give back the information to the AMF
5. The AMF request the SMF selection subcription data
6. Then the UDM will respond with the requested data
7. The AMF will request the UE context in the SMF data
8. Then the UDM will respond with the requested data
9. The AMF will create the UE context for the User
10. UDM inform the old AMF, that it is no longer serving the user
11. The old AMF inform the SMF that it is no longer associated with the PDU Session
12. Then the old AMF will delete the UE context

## Register and Update policy association with the PCF
1. The AMF will select a PCF and contact the PCF to create policy association and retrieve UE policy and Access and Mobility control policy
2. The PCF responds with the policy association information. 
3. The PCF registers for events like "Location Report", "Registration State Report" and "Communication Failure Report". 
4. The AMF responds with "201 Created" to signal successful subscription. 
5. The Old AMF requests that the policy association is deleted as the corresponding UE context is terminated. 
6. PCF signals the successful delete with the "204 No Content" HTTP response code. 

## Setup the UPF (User Plane Function)
1. The new AMF will send a request to the SMF to setup a new session
2. The SMF will allocate an IP address for the UE
3. The SMF will allocate a TEID (Tunnel Endpoint Identifier) for the gNodeB that will be use when sending uplink GTP PDUs to the UPF
4. The SMF will select a UPF for the user
5. The PFCP session modification request is sent from the SMF to the UPF
6. The UPF will start receiving downlink data from the internet that is destined to the UE
7. The data that is destined for the UE is being hold in the UPF buffer
8. The UPF will give response to the SMF for the session modification request
9. The SMF inform the AMF that the session management context has been updated
10. The AMF will allocate a "AMF UE NGAP ID" that will be use to identify the UE context by the gNodeB on the AMF
11. The AMF will initiates a session setup with the gNB with a NAS message which contains one or more PDU session setup request, along with the PDU Session ID, TEID for every PDU session and the "AMF UE NGAP ID", "UE Aggregate Maximum Bit Rate", UE security capabilities and security key.


## 5G NR AS Security Procedure
1. a K-gNB key is derived by the UE and AMF from K-AMF. 
2. The UE will Configure lower layers to apply SRB integrity protection using the indicated algorithm and the K-RRC-int key immediately. 
3. The UE security mode complete message confirms the successful completion of the security mode command. This message is integrity protected but not ciphered. Ciphering will start immediately after sending this message. 
4. The UE will Configure lower layers to apply SRB ciphering using the indicated algorithm, the K-RRC-enc key after completing the procedure. The Security Mode Complete message is not ciphered. 
:::spoiler
The K-RRC-enc key is a security used for encryption of RRC(Radio Resource Control) messages
:::

## 5G NR RRC Reconfiguration
1. The gNodeB sent a RRC Reconfiguration message to the UE for setting up radio bearers, setting up a secondary cell and initiate UE measurements. 
2. The UE will Confirm the successful completion of an RRC connection reconfiguration. 
3. The gNodeB will Allocate the TEID that the UPF will use to send downlink data to the gNB. 
4. The gNB signals the AMF the successful setup of PDU sessions. The message also carries the Downlink TEID that should be used (specified per PDU session). 
5. The UE signals the completion of the registration via the "Registration Complete" message to the AMF. 

## Start Downlink and Uplink Data Transfer
1. The gNB will start sending the UE data to the Uplink TEID. 
2. The UPF starts sending the Ue data to the Internet. 
3. The AMF modifies the Session Management Context based on the updates from the gNB, by sending a PDU Session Update to the SMF
4. The SMF control plane signals session updates to the UPF data plane. 
5. UPF  stop the data buffering as a downlink path has been setup. 
6. The UPF sends the buffered data to the gNB using the Downlink TEID for the PDU session. 
7. The UPF data plane responds back to SMF control plane.
8. The SMF notifies the AMF that session management context update is complete. 

### SMF Interactions in the 5G Standalone Access Registration
## Register with the UDM and obtain subscription data
1. The Old AMF send a PDUSession_Release request to the SMF

### Setting up the UPF
1. The new AMF will send a PDUSession_Update request to the SMF
2. The SMF will allocate the UE IP address
3. The SMF will allocate the PDU Session Uplink TEID
4. The SMF will select an UPF
5. The SMF will send a PFCP Session Modification Request to the UPF
6. The UPF will give response to the SMF
7. The SMF will give response to the new AMF a PDUSession_Update response

### Start Donwlink and Uplink Data transfer
1. The New AMF will send a PDUSession_Update request to the SMF
2. The SMF will signal the UPF with PFCP Session Modification Request
3. The UPF will send a PFCP Session modification Response
4. The SMF will give a PDUSession_Update response to the new AMF 