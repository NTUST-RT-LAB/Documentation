# AMF (Access and Mobility Management Function)
## N50 implementation in free5gc
N50 is an interface that responsible for transferring message between CBCF and AMF. We will try to implement the N50 in free5GC

* Run free5gc
```
cd free5gc
./run.sh
```
![](https://hackmd.io/_uploads/BkqIGeKS2.png)

* Send HTTP Request to N50 (Non UE N2 Transfer Message)
```
curl -X POST -H "Content-Type: application/json" 127.0.0.18:8000/namf-comm/v1/non-ue-n2-messages/transfer
```
![](https://hackmd.io/_uploads/B1oM7xYS3.png)

![](https://hackmd.io/_uploads/HkVDQltH2.png)

:::danger
We can see that the response is empty and the free5gc log shows that the Handle for Non UE N2 Message Transfer is not implemented
:::

* Trace the code of Non UE N2 Message Transfer
```
cd NFs/amf/internal/sbi/communication/
nano api_non_uen2_messages_collection_document.go
```
![](https://hackmd.io/_uploads/ryQBNeFS2.png)

![](https://hackmd.io/_uploads/HJL-ExKB2.png)
:::info
We can see that the API is not yet implemented (only give response message like the pictures above)
:::


### Trace the code in the OpenAPI github of free5gc
In free5GC, there is a repository called OpenAPI, that contains APIs that is used for the free5gc, including the api_non_uen2_messages_collection_document, it can be seen [here](https://github.com/free5gc/openapi/blob/main/Namf_Communication/api_non_uen2_messages_collection_document.go) 

![](https://hackmd.io/_uploads/ry2fWQCU2.png)

:::info
After tracing the code, there is few points:
* The function will create path and map variables for the params, like header and query
* After that, the function will call the `openApi.PrepareRequest()`, the function is to create a HTTP Request based on all the params before
* If the HTTP Request is created, it will call another function called `openApi.CallApi()`, the function is to call the API URL with the HTTP Request created before
* If the HTTP Request is success, it will check the response code and process the return value based on the response code
* In conclusion, the api_non_uen2_messages_collection_document in OpenAPI is just a function to create a standarized HTTP Request, the function for the AMF to handle the request is not available
:::

## Add database functionality in AMF
We will try to add database functionality to AMF, so that the AMF will be able to save the message into database. The database that will be used is MongoDB
### Add Database Functionality to NonUeN2MessageTransfer
* Enter the MongoDB shell:
```
mongo
```
![](https://hackmd.io/_uploads/HJ624fuDh.png)


* Create new collection in MongoDB called AMFNonUeN2MessageTransfer
```
db.createCollection("AMFNonUeN2MessageTransfer")
```
![](https://hackmd.io/_uploads/HyeBSGuwh.png)

* Exit the MongoDB and edit the NonUeN2Transfer Message
```
exit
cd free5gc/NFs/amf/internal/sbi/communication/
nano api_non_uen2_messages_collection_document.go
```
![](https://hackmd.io/_uploads/rJARBMdPh.png)

![](https://hackmd.io/_uploads/BknlLGuw3.png)

* Add the insertToDatabase function:
![](https://hackmd.io/_uploads/r1FLDXOPn.png)

:::spoiler The code:
```
func insertToDatabase(message models.NonUeN2MessageTransferRequest) {
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Connected to MongoDB!")
	collection := client.Database("local").Collection("AMFNonUeN2MessageTransfer")
	insertResult, err := collection.InsertOne(context.TODO(), message)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Inserted document ID: %v\n", insertResult.InsertedID)

}
```
:::

* Build the AMF
```
cd ~/free5gc 
make amf
```
![](https://hackmd.io/_uploads/HkZpv7uw2.png)

* Run free5gc
```
./run.sh
```
![](https://hackmd.io/_uploads/rJDXdXuvh.png)

* Open a new terminal and run CBCF
```
cd ~/cbcftest
./CBCF
```
![](https://hackmd.io/_uploads/rkXBd7_w3.png)

* Access the endpoint using CURL with data
```
curl -X POST -d "ratSelector=NR&tac=tac&mnc=30&mcc=20" http://localhost:8080/
```
![](https://hackmd.io/_uploads/SknIdQuDn.png)

* Output in free5gc
![](https://hackmd.io/_uploads/rycuOmOD3.png)

* Enter MongoDB again to see the data
```
mongo
db.AMFNonUeN2MessageTransfer.find().pretty()
```
![](https://hackmd.io/_uploads/Bk92_XOw3.png)

:::success
The message is successfully inserted into the database
:::

### Add Database Functionality to NonUeN2InfoSubscribe
* Enter the MongoDB shell:
```
mongo
```
![](https://hackmd.io/_uploads/BJCHvghu2.png)


* Create new collection in MongoDB called AMFNonUeN2MessageSubscriptions
```
db.createCollection("AMFNonUeN2MessageSubscriptions")
```
![](https://hackmd.io/_uploads/r1Ytvlnun.png)

* Exit the MongoDB and edit the NonUeN2Transfer Message
```
exit
cd free5gc/NFs/amf/internal/sbi/communication/
nano api_non_uen2_messages_subscriptions_collection_document.go.go
```

![](https://hackmd.io/_uploads/SyuyOx2dh.png)

* Add the insertToDatabase function:
![](https://hackmd.io/_uploads/rksSOehd2.png)


:::spoiler The code:
``` go
func insertToSubscriptionDatabase(message models.NonUeN2InfoSubscriptionCreateData) {
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Connected to MongoDB!")
	collection := client.Database("local").Collection("AMFNonUeN2MessageSubscriptions")
	insertResult, err := collection.InsertOne(context.TODO(), message)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Inserted document ID: %v\n", insertResult.InsertedID)

}
```
:::

* Build the AMF
```
cd ~/free5gc 
make amf
```
![](https://hackmd.io/_uploads/BJFuuehu3.png)

* Run free5gc
```
./run.sh
```
![](https://hackmd.io/_uploads/S1cnul3O3.png)

* Open a new terminal and run CBCF
```
cd ~/cbcftest
./CBCF
```
![](https://hackmd.io/_uploads/S1YRdlndh.png)

* Access the endpoint using CBE simulator
```
curl -X POST -d "ratSelector=NR&tac=tac&mnc=30&mcc=20" http://localhost:8080/
```
![](https://hackmd.io/_uploads/ryzbYg2_n.png)

* Output in free5gc
![](https://hackmd.io/_uploads/HJHGKe2un.png)

* Enter MongoDB again to see the data
```
mongo
db.AMFNonUeN2MessageSubscriptions.find().pretty()
```
![](https://hackmd.io/_uploads/SJVwFe2dh.png)

![](https://hackmd.io/_uploads/rkrLKghO3.png)

:::success
The message is successfully inserted into the database
:::

### Add Database Functionality to NonUeN2InfoSubscribe
This function will unsubscribe, so it has to delete the message in the subscription database or the AMFNonUeN2MessageSubscriptions

* Edit the NonUeN2Transfer Message
```
cd free5gc/NFs/amf/internal/sbi/communication/
nano api_non_uen2_message_notification_individual_subscription_document.go
```

![](https://hackmd.io/_uploads/rJq75x3dh.png)

* Add the deleteObjectByID function:
![](https://hackmd.io/_uploads/SJ6Scg2O3.png)


:::spoiler The code:
``` go
func deleteObjectByID(id string) error {
	// Convert the string ID to MongoDB's ObjectId type
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Connected to MongoDB!")
	collection := client.Database("local").Collection("AMFNonUeN2MessageSubscriptions")
	objectID, err := primitive.ObjectIDFromHex(id)
	if err != nil {
		return fmt.Errorf("failed to convert ID to ObjectId: %v", err)
	}

	// Create a filter to match the document with the given ID
	filter := bson.M{"_id": objectID}

	// Delete the document matching the filter
	result, err := collection.DeleteOne(context.Background(), filter)
	if err != nil {
		fmt.Println("failed to delete document: %v", err)
	}

	// Check the result to see if any documents were deleted
	if result.DeletedCount == 0 {
		fmt.Println("No documents were deleted")
	}

	fmt.Printf("Deleted %d document(s) with ID %s", result.DeletedCount, id)
	return nil
}
```
:::

:::info
This function will find the subscription id, by the object id from the params, and after that it will delete it
:::
* Build the AMF
```
cd ~/free5gc 
make amf
```
![](https://hackmd.io/_uploads/BJFuuehu3.png)

* Run free5gc
```
./run.sh
```
![](https://hackmd.io/_uploads/S11oqxhdh.png)

* Open a new terminal and run CBCF
```
cd ~/cbcftest
./CBCF
```
![](https://hackmd.io/_uploads/Sk82qx2dn.png)

* Try to delete the previously created subscriptions using cURL. The previously created subscriptions object id is 649e78ecb6810af082890cf1
```
curl -X DELETE 127.0.0.1:8080/unsubscribe/649e78ecb6810af082890cf1
```
![](https://hackmd.io/_uploads/ryYEsxnOn.png)

* Output in free5gc
![](https://hackmd.io/_uploads/BkTLslhdh.png)

* Enter MongoDB again to see the data with the object id 649e78ecb6810af082890cf1
```
mongo
db.AMFNonUeN2MessageSubscriptions.findOne({ _id: ObjectId("649e78ecb6810af082890cf1") })
```
![](https://hackmd.io/_uploads/HyI-3e2_h.png)

:::success
It returns null that means the message is successfully deleted from the database
:::

## Implement NGAP Protocol in AMF
### Trying to send the message to RAN
In this section, we will try to send NGAP message to RAN, the RAN that will be used is UERANSIM, the documentation for UERANSIM is located [here](https://hackmd.io/@joshevan/UERANSIM-Installation). To try to send the NGAP message, we will use this code:
``` go=
amfSelf := amf_context.AMF_Self()
ue, ok := amfSelf.AmfUeFindByUeContextID("imsi-208930000000003")
if !ok {
	fmt.Println("UE Not found")
    }
var targetValue *amf_context.RanUe
for key, value := range ue.RanUe {
	if key == "3GPP_ACCESS" {
		targetValue = value
		break
	}
}
ngap_message.SendToRanUe(targetValue, message.BinaryDataN2Information)
```
In this code, first the AMF will find the UE by context id, or by the IMSI in this case. After the UE found, we search for the RAN and we send it by the `sendToRanUe` function. That function receive two parameters, the RanUE and the bytes of information. We will try to send the Message Binary Data N2 Information directly because it is a bytes and it contains the write replace warning request message. After that we will try it

Steps to try it:
* Build the AMF and run the Free5GC:
```
make amf
./run.sh
```
![](https://hackmd.io/_uploads/SyGTOc7Y2.png)

* Run the UERANSIM VM, and run the RAN and the UE
```
cd ~/UERANSIM
build/nr-gnb -c config/free5gc-gnb.yaml
```
![](https://hackmd.io/_uploads/r1dou9Xtn.png)

```
cd ~/UERANSIM
sudo build/nr-ue -c config/free5gc-ue.yaml
```
![](https://hackmd.io/_uploads/SJ_ZK5QYh.png)

* Run the CBCF
```
cd ~/cbcftest
./CBCF
```
![](https://hackmd.io/_uploads/SJwrF9mtn.png)

* Run the CBE Simulator
```
cd ~/cbcftest
python3 CBE.py --ratSelector=NR --tac=tac --mnc=93 --mcc=208 --messageId=1 --n2InformationClass=PWS
```
![](https://hackmd.io/_uploads/SkYlcc7Y2.png)

* Output in RAN (UERANSIM)

![](https://hackmd.io/_uploads/H16Qq57th.png)

It says that error because of APER decoding failed, it is because the bytes is send directly without any handling, we must build the NGAP package properly to be able to send it into the RAN

### Trying to build Write Replace Warning Request Message and send to RAN
In this section, we will try to build Write Replace Warning Request, we will try to use other function as a reference to build the message. The code is located in the NGAP/message folder in AMF, in the `build.go` file

:::spoiler The code
``` go=
func BuildWriteReplaceWarningRequest() ([]byte, error) {
	var pdu ngapType.NGAPPDU
	pdu.Present = ngapType.NGAPPDUPresentInitiatingMessage
	pdu.InitiatingMessage = new(ngapType.InitiatingMessage)

	initiatingMessage := pdu.InitiatingMessage
	initiatingMessage.ProcedureCode.Value = ngapType.ProcedureCodeWriteReplaceWarning
	initiatingMessage.Criticality.Value = ngapType.CriticalityPresentReject

	initiatingMessage.Value.Present = ngapType.InitiatingMessagePresentWriteReplaceWarningRequest
	initiatingMessage.Value.WriteReplaceWarningRequest = new(ngapType.WriteReplaceWarningRequest)

	WriteReplaceWarningRequest := initiatingMessage.Value.WriteReplaceWarningRequest
	WriteReplaceWarningRequestIEs := &WriteReplaceWarningRequest.ProtocolIEs

	ie := ngapType.WriteReplaceWarningRequestIEs{}
	ie.Id.Value = ngapType.ProtocolIEIDNumberOfBroadcastsRequested
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentNumberOfBroadcastsRequested
	ie.Value.NumberOfBroadcastsRequested = new(ngapType.NumberOfBroadcastsRequested)

	numberOfBroadcastsRequested := ie.Value.NumberOfBroadcastsRequested
	numberOfBroadcastsRequested.Value = 10
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)
	return ngap.Encoder(pdu)
}
```
:::

The code will build the PDU of write replace warning request as initiating message. We set the procedure code to be the procedure code of write replace warning request message, which is 51, and the value present of the initiating message to be the value of the write replace warning request, which is 18. After that we will call this function in the communication and we will send it to the ran.
Steps to try the code:

* Build the AMF and run the Free5GC:
```
make amf
./run.sh
```


* Run the UERANSIM VM, and run the RAN
```
cd ~/UERANSIM
build/nr-gnb -c config/free5gc-gnb.yaml
```
![](https://hackmd.io/_uploads/BJl3oAdK3.png)


* Run the CBCF
```
cd ~/cbcftest
./CBCF
```
![](https://hackmd.io/_uploads/rJoCiAOYn.png)

* Run the CBE Simulator
```
cd ~/cbcftest
python3 CBE.py --ratSelector=NR --tac=tac --mnc=93 --mcc=208 --messageId=1 --n2InformationClass=PWS
```
![](https://hackmd.io/_uploads/r1Of2C_Yn.png)

* Output in RAN (UERANSIM)

![](https://hackmd.io/_uploads/BJcQ2RuYh.png)

The output says that the NGAP is not handled, and the code is 18, it is the same with the Write Replace Warning Request initiaiting value, which means that Write Replace Warning Request has not been implemented in UERANSIM


* Output in Wireshark UERANSIM

![](https://hackmd.io/_uploads/B1BkDNKKn.png)

The output in the wireshark shows that the NGAP package is successfully received in the UERANSIM, and the package is also identified as Write Replace Warning Request, and also the information in the package is successfully received, it means that the package has been successfully received and identified, but the UERANSIM does not have the handle for it. 

### Use other ran
Previously, the message is successfully received in the UERANSIM, and the package is also identified as Write Replace Warning Request, but the UERANSIM does not have the handle for it, so we will try to use other RAN that has the handler for Write Replace Warning Request message. The type of RAN that will be used is Nokia 474021A ASIK AirScale.
The steps for testing is the same as above, we run the Free5GC, connect the RAN to Free5GC, run the CBCF, and send the message via CBE simulator. Below is the result in wireshark:

![](https://hackmd.io/_uploads/r1dcW7Ctn.png)

Packet 24:
![](https://hackmd.io/_uploads/rkN2-XAKh.png)

Packet 25:
![](https://hackmd.io/_uploads/r1E0ZQRt3.png)

The message with all the information is successfully received in the RAN, but the next packet is it sends the error indication with the cause of the error is message not compatible with receiver state, it is probably because it has the handler for Write Replace Warning Request message, but it does not handle all kind of message. For the next step, we will try to trace the RAN to see which kind of message it support and then modify the AMF to match it.

### Build again the message
After reading some documentation, the CMAS message should contain some parameters:
* Message Identifier: will be set to 4370 (CMAS Presidential Level Alerts)
* Serial Number: will be set to 5940
* Warning Area List: will be set to TAI with mcc 466, mnc 66, TAC 437d58 (The TAI of the tested gNB)
* Repetition Period: will be set to 240
* Number of Broadcasts Requested: will be set to 1
* Data Coding Scheme: will be set to 01 (English Language)
* Warning Message Contents: will be set to "This is an example warning message for testing purposes.", but the message should be encoded to GSM 7 bit, so that it is understandable by the gNB
* Conccurent Warning Message Indication: will be set to 0 (true)

After building the package, the package will be sent to gNB, the output in the Wireshark:
![](https://hackmd.io/_uploads/H19T4Ow9h.png)

![](https://hackmd.io/_uploads/SJWR4_wch.png)

![](https://hackmd.io/_uploads/SkoAV_vc3.png)

The gNB sends back the Write Replace Warning Response, it indicates that the Write Replace Warning Request is succesffully sent and successfully broadcasted, the output in UE:
![](https://hackmd.io/_uploads/S1EVS_Pcn.png)

The PWS message is successfully shown and the message is also according to the one we set, so the PWS procedure is already succeded. For the next part, we will make the message according to data from the CBCF because now the message is still hardcoded

### Build message using parameters from CBCF
Before, the message is still hardcoded, we will try to modify the parameters so that it takes input from CBCF, just like in the real cases. The code:

``` go=
func BuildWriteReplaceWarningRequest(keyValueN2Information map[string]string) ([]byte, error) {
	var pdu ngapType.NGAPPDU
	var err error
	pdu.Present = ngapType.NGAPPDUPresentInitiatingMessage
	pdu.InitiatingMessage = new(ngapType.InitiatingMessage)

	initiatingMessage := pdu.InitiatingMessage
	initiatingMessage.ProcedureCode.Value = ngapType.ProcedureCodeWriteReplaceWarning
	initiatingMessage.Criticality.Value = ngapType.CriticalityPresentReject

	initiatingMessage.Value.Present = ngapType.InitiatingMessagePresentWriteReplaceWarningRequest
	initiatingMessage.Value.WriteReplaceWarningRequest = new(ngapType.WriteReplaceWarningRequest)

	WriteReplaceWarningRequest := initiatingMessage.Value.WriteReplaceWarningRequest
	WriteReplaceWarningRequestIEs := &WriteReplaceWarningRequest.ProtocolIEs

	ie := ngapType.WriteReplaceWarningRequestIEs{}

	ie.Id.Value = ngapType.ProtocolIEIDMessageIdentifier
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentMessageIdentifier
	ie.Criticality.Value = ngapType.CriticalityPresentReject
	ie.Value.MessageIdentifier = new(ngapType.MessageIdentifier)
	messageIdentifier := ie.Value.MessageIdentifier
	messageIdentifier.Value = *new(aper.BitString)
	messageIdentifier.Value.Bytes, err = hex.DecodeString(keyValueN2Information["messageIdentifier"])
	if err != nil {
		logger.NgapLog.Error(err)
	}
	messageIdentifier.Value.BitLength = 16
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDSerialNumber
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentSerialNumber
	ie.Criticality.Value = ngapType.CriticalityPresentReject
	ie.Value.SerialNumber = new(ngapType.SerialNumber)
	serialNumber := ie.Value.SerialNumber
	serialNumber.Value = *new(aper.BitString)
	serialNumber.Value.Bytes, err = hex.DecodeString(keyValueN2Information["serialNumber"])
	if err != nil {
		logger.NgapLog.Error(err)
	}
	serialNumber.Value.BitLength = 16
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDWarningAreaList
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentWarningAreaList
	ie.Criticality.Value = ngapType.CriticalityPresentIgnore
	ie.Value.WarningAreaList = new(ngapType.WarningAreaList)
	warningAreaList := ie.Value.WarningAreaList
	warningAreaList.Present = ngapType.WarningAreaListPresentTAIListForWarning
	warningAreaList.TAIListForWarning = new(ngapType.TAIListForWarning)
	taiListForWarning := warningAreaList.TAIListForWarning
	taiListForWarning.List = make([]ngapType.TAI, 0)
	context.AMF_Self().AmfRanPool.Range(func(key, value interface{}) bool {
		tai := ngapType.TAI{}
		plmnId := models.PlmnId{}
		amfRan := value.(*context.AmfRan)
		for i := 0; i < len(amfRan.SupportedTAList); i++ {
			plmnId.Mcc = amfRan.SupportedTAList[i].Tai.PlmnId.Mcc
			plmnId.Mnc = amfRan.SupportedTAList[i].Tai.PlmnId.Mnc
			tac := amfRan.SupportedTAList[i].Tai.Tac
			tai.PLMNIdentity = ngapConvert.PlmnIdToNgap(plmnId)
			tai.TAC = *new(ngapType.TAC)
			tai.TAC.Value = *new(aper.OctetString)
			tai.TAC.Value, err = hex.DecodeString(tac)
			taiListForWarning.List = append(taiListForWarning.List, tai)
		}
		return true
	})

	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDRepetitionPeriod
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentRepetitionPeriod
	ie.Criticality.Value = ngapType.CriticalityPresentReject
	ie.Value.RepetitionPeriod = new(ngapType.RepetitionPeriod)
	repetitionPeriod := ie.Value.RepetitionPeriod
	repetitionPeriodInt, err := strconv.Atoi(keyValueN2Information["repetitionPeriod"])
	repetitionPeriod.Value = int64(repetitionPeriodInt)
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDNumberOfBroadcastsRequested
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentNumberOfBroadcastsRequested
	ie.Criticality.Value = ngapType.CriticalityPresentReject
	ie.Value.NumberOfBroadcastsRequested = new(ngapType.NumberOfBroadcastsRequested)
	numberOfBroadcastsRequested := ie.Value.NumberOfBroadcastsRequested
	numberOfBroadcastsRequestedInt, err := strconv.Atoi(keyValueN2Information["numberOfBroadcastsRequested"])
	numberOfBroadcastsRequested.Value = int64(numberOfBroadcastsRequestedInt)
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDDataCodingScheme
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentDataCodingScheme
	ie.Criticality.Value = ngapType.CriticalityPresentIgnore
	ie.Value.DataCodingScheme = new(ngapType.DataCodingScheme)
	dataCodingScheme := ie.Value.DataCodingScheme
	dataCodingScheme.Value = *new(aper.BitString)
	dataCodingScheme.Value.Bytes, err = hex.DecodeString(keyValueN2Information["dataCodingScheme"])
	if err != nil {
		logger.NgapLog.Error(err)
	}
	dataCodingScheme.Value.BitLength = 8
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDWarningMessageContents
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentWarningMessageContents
	ie.Criticality.Value = ngapType.CriticalityPresentIgnore
	ie.Value.WarningMessageContents = new(ngapType.WarningMessageContents)
	warningMessageContents := ie.Value.WarningMessageContents
	message := keyValueN2Information["warningMessageContents"]
	var messageBytes []byte
	var bytesLength []byte
	if keyValueN2Information["dataCodingScheme"] == "00" {
		tpdus, _ := sms.Encode([]byte(message))
		for _, p := range tpdus {
			messageBytes, _ = p.MarshalBinary()
			// send binary TPDU...
		}
		messageBytes = messageBytes[7:]
	}
	if keyValueN2Information["dataCodingScheme"] == "48" {
		encodedMessage := fmt.Sprintf("%04x", utf16.Encode([]rune(message)))
		encodedMessage = strings.Replace(encodedMessage[1:len(encodedMessage)-1], " ", "", -1)
		messageBytes, err = hex.DecodeString(encodedMessage)
		fmt.Println(messageBytes)
	}
	pagesNumber, err := hex.DecodeString("01")
	messageLength := strconv.FormatInt(int64(len(messageBytes)), 16)
	if len(messageLength)%2 == 1 {
		bytesLength, err = hex.DecodeString("0" + messageLength)
	} else {
		bytesLength, err = hex.DecodeString(messageLength)
	}
	warningMessageContents.Value = append(pagesNumber, messageBytes...)
	for len(warningMessageContents.Value) < 83 {
		warningMessageContents.Value = append(warningMessageContents.Value, 0x00)
	}
	warningMessageContents.Value = append(warningMessageContents.Value, bytesLength...)
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)

	ie.Id.Value = ngapType.ProtocolIEIDConcurrentWarningMessageInd
	ie.Value.Present = ngapType.WriteReplaceWarningRequestIEsPresentConcurrentWarningMessageInd
	ie.Criticality.Value = ngapType.CriticalityPresentReject
	ie.Value.ConcurrentWarningMessageInd = new(ngapType.ConcurrentWarningMessageInd)
	concurrentWarningMessageInd := ie.Value.ConcurrentWarningMessageInd
	concurrentWarningMessageInd.Value = *new(aper.Enumerated)
	concurrentWarningMessageInd.Value = 0
	WriteReplaceWarningRequestIEs.List = append(WriteReplaceWarningRequestIEs.List, ie)
	return ngap.Encoder(pdu)
}

```

Command to send the message in CBCF:
```
python3 CBE.py --ratSelector=NR --tac=437d58 --mnc=66 --mcc=466 --messageId=1 --n2InformationClass=PWS --repetitionPeriod=240 --numberOfBroadcastsRequested=3 --warningMessageContents="Example Message"
```

Output in wireshark:
![](https://hackmd.io/_uploads/rkX613353.png)


The input from CBCF is successfully inserted as protocol IE (information element). It means that the Write Replace Warning Request package has used the data from the CBCF, Output in UE:
![](https://hackmd.io/_uploads/HkP8x22q3.jpg)

The PWS message is successfully shown and the message is also according to the CBCF, I also added the timestamp to show the time of the message. The warning message content is still in English language, we will try to modify it so that the warning message content can be in Taiwanese language
