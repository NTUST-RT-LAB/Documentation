# CBCF (Cell Broadcast Centre Function)
CBCF (Cell Broadcast Centre Function) is a network function in the 5G core network that responsible for all kind of broadcast message, including PWS (Public Warning System). CBCF is the same with CBC in 5G, but already implemented 5G core concept, which is the SBA (Service Based Arhitecture). The message delivery procedure using these programs can be seen on this [link](https://hackmd.io/@joshevan/3GPP-TS-23041#9A-Warning-Message-Delivery-using-5G-Service-Based-Interface)

## Go Program to test AMF N2 interface
We will create a new folder called 'cbcftest' to create all the program to test AMF N2 interface. ALl the program is stored in this [repository](https://github.com/Joshevanch/CBCF)
* Create the folder
```
mkdir cbcftest
cd cbcftest
```
![](https://hackmd.io/_uploads/S1ohZsWU2.png)

### Non UE N2 Transfer Message
To use Non UE N2 Transfer Message in AMF we will create a go program to acts as HTTP client and send a HTTP Request to use Non UE N2 transfer Message in AMF.
* Create the code
```
nano NonUeN2MessageRequest.go
```
![](https://hackmd.io/_uploads/SkP0-oZI2.png)


![](https://hackmd.io/_uploads/r1Vyx3sH2.png)

:::spoiler The code:
```go=
package main

import (
	"bytes"
	"fmt"
	"net/http"
	"io/ioutil"
	"encoding/json"
	"github.com/free5gc/openapi/models"
)

func transfer(m map[string]string) {
	// Specify the URL you want to send the request to
	url := "http://127.0.0.18:8000/namf-comm/v1/non-ue-n2-messages/transfer/"
	// Create the request body
	message := models.NonUeN2MessageTransferRequest{}
	jsonString := []byte(`{
		"jsonData": {
		  "taiList": [
			{
			  "tac": "",
			  "plmnId": {
				"mnc": "",
				"mcc": ""
			  }
			}
		  ],
		  "ratSelector": "PWS",
		  "ecgiList": [
			{
			  "eutraCellId": "",
			  "plmnId": {
				"mnc": "",
				"mcc": ""
			  }
			}
		  ],
		  "ncgiList": [
			{
			  "nrCellId": "",
			  "plmnId": {
				"mnc": "",
				"mcc": ""
			  }
			}
		  ],
		  "globalRanNodeList": [
			{
			  "gNbId": {
				"bitLength": 24,
				"gNBValue": ""
			  },
			  "plmnId": {
				"mnc": "",
				"mcc": ""
			  },
			  "n3IwfId": "",
			  "ngeNbId": ""
			}
		  ],
		  "n2Information": {
			"n2InformationClass": "",
			"smInfo": {
			  "subjectToHo": false,
			  "pduSessionId": 29,
			  "n2InfoContent": {
				"ngapData": {
				  "contentId": "contentId"
				},
				"ngapIeType": "",
				"ngapMessageType": 32
			  },
			  "sNssai": {
				"sd": "sd",
				"sst": 32
			  }
			},
			"ranInfo": {
			  "n2InfoContent": {
				"ngapData": {
				  "contentId": "contentId"
				},
				"ngapIeType": "",
				"ngapMessageType": 32
			  }
			},
			"nrppaInfo": {
			  "nfId": "nfId",
			  "nrppaPdu": {
				"n2InfoContent": {
				  "ngapData": {
					"contentId": "contentId"
				  },
				  "ngapIeType": "",
				  "ngapMessageType": 32
				}
			  }
			},
			"pwsInfo": {
			  "messageIdentifier": 0,
			  "serialNumber": 0,
			  "pwsContainer": {
				"n2InfoContent": {
				  "ngapData": {
					"contentId": "contentId"
				  },
				  "ngapIeType": "",
				  "ngapMessageType": 51
				},
				"sendRanResponse": true,
				"omcId": true
			  }
			}
		  },
		  "supportedFeatures": ""
		}
	  }`)
	json.Unmarshal(jsonString, &message)
	if (m["ratSelector"] == "NR"){
		message.JsonData.RatSelector = models.RatSelector_NR
	}
	if (m["ratSelector"] == "E-UTRA"){
		message.JsonData.RatSelector = models.RatSelector_E_UTRA
	}
	(*message.JsonData.TaiList)[0].PlmnId.Mcc = m["mcc"]
	(*message.JsonData.TaiList)[0].PlmnId.Mnc = m["mnc"]
	(*message.JsonData.TaiList)[0].Tac = m["tac"]
	fmt.Printf("%+v", (*message.JsonData.TaiList)[0].PlmnId)
	jsonString, err := json.Marshal(message)
	req, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonString))
    req.Header.Set("X-Custom-Header", "myvalue")
    req.Header.Set("Content-Type", "application/json")
    client := &http.Client{}
    response, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error sending request: %s\n", err)
		return
	}
	defer response.Body.Close()

	body, err := ioutil.ReadAll(response.Body)
	if err != nil {
		fmt.Printf("Error reading response: %s\n", err)
		return
	}
	fmt.Println(string(body))
}

```

:::
:::info
The JSON is hardcoded in the code and will be sent as a body in HTTP Request to Non UE N2 transfer Message in AMF.
If we see the OpenAPI, the schema of the JSON is described in the models.NonUeN2MessageTransferRequest, which is the same with N2InformationTransferReqData that can be seen at [3GPP SPEC TS 29 518](https://www.etsi.org/deliver/etsi_ts/129500_129599/129518/15.06.00_60/ts_129518v150600p.pdf)
![](https://hackmd.io/_uploads/HyeiUojrn.png)
:::
:::info
The complete details of the N2InformationTransferReqData:
* taiList: List of TAI (Tracking Area Identity), a numeric identifier that represents a specific tracking area within the network. 
    * TAI is identified by TAC (Tracking Area Code) and PLMN (Public Land Mobile Network)
        * PLMN is identified by MNC (Mobile Network Code) and MCC (Mobile Country Code)
* ratSelector: RAT (Radio Access Technology) selector, enumeration to choose between E-UTRA (4G) and NG (5G)
* ecgiList: List of ECGI (E-UTRAN Cell Global Identifier), a numeric identifier that represents E-UTRA Cells. 
    * ECGI is identified by eutraCellId, a numeric identifier that represents a specific cell within the network and PLMN (Public Land Mobile Network)
* ncgiList: List of NCGI (NG Cell Global Identifier), a numeric identifier that represents NG Cells. 
    * NCGI is identified by nrCellId, a numeric identifier that represents a specific cell within the network and PLMN (Public Land Mobile Network)
* globalRanNodeList: List of RAN Nodes, 
    * Identified by gnbID, the global Node base station ID, PLMN (Public Land Mobile Network), n3IwfId (Non 3GPP Inter Working Function), and ngeNbId (New Generation enhanced Node-B) 
* n2Information: The information of the N2 message, consists of:
    * n2InformationClass: enumeration to choose class of the N2 message, it can be PWS, SM, RAN, NRPPa, PWS-BCAL (PWS Brodcast Completed Area List) or PWS-RF (PWS Restart Indication or Failure Indication)
    * smInfo: Session Management Information if N2 information Class is SM. SMInfo has few informations:
        * subjectToHo indicates whether the transfer required handover or not 
        * pduSessionId indicates the PDU Session identity
        * n2InfoContent indicates the NGAP information, the content id, the message type, and the IE type
        * sNssai shall be present if network slice information to be transferred for session management
    * ranInfo: RAN Information if N2 information Class is RAN, it only has the n2Infocontent
    * nrppaInfo: NRPPA (NR Positioning Protocol A) Information if N2 information Class is RAN, it contains information of nfId (Network Function ID) and n2InfoContent
    * pwsInfo: PWS Information if N2 information Class is PWS. PWS has few informations:
        * messageIdentifier: The id of the message (The default value is 0)
        * serialNumber: Particular message from the source (The default value is 0)
        * pwsContainer: The container of n2InfoContent (The ngapMessageType should be 51 to indicate Write-Replace-Warning (from 3GPP Spec 38.413 about NG-AP))
        * sendRanResponse: determine to receive RAN Response from AMF or not
        * omcId: The identity of Operation and Maintenance Center

The mandatory parameters is the n2Information, the rest is optional
:::

* Modify NonUeN2MessagesCollection to print request message
```
cd NFs/amf/internal/sbi/communication/
nano api_non_uen2_messages_collection_document.go
make amf
```
![](https://hackmd.io/_uploads/BJG1y3jr2.png)

![](https://hackmd.io/_uploads/SJxtg3jrn.png)

![](https://hackmd.io/_uploads/BJl7e2sHh.png)

* Run Free5GC, open a new terminal and send the request
```
./run.sh
```
![](https://hackmd.io/_uploads/r17aghjH2.png)

```
go run NonUeN2MessageRequest.go
```
![](https://hackmd.io/_uploads/BJf9fs-Ln.png)


![](https://hackmd.io/_uploads/H1SZk2jrn.png)
 
:::success
The message is successfully received in the AMF
:::


### Non UE N2 Info Subscribe:
To use Non UE N2 Info Subscribe in AMF we will create a go program to acts as HTTP client and send a HTTP Request to use Non UE N2 info unsubscribe in AMF.
* Create the code
```
nano NonUeN2InfoUnsubscribe.go
```
![](https://hackmd.io/_uploads/SyYb7iW83.png)

![](https://hackmd.io/_uploads/r1Vyx3sH2.png)

:::spoiler The code:
```go=
package main

import (
	"bytes"
	"fmt"
	"net/http"
	"io/ioutil"
)

func main() {
	// Specify the URL you want to send the request to
	url := "http://127.0.0.18:8000/namf-comm/v1/non-ue-n2-messages/subscriptions"

	// Create the request body
	jsonString := []byte(`{
		"globalRanNodeList": [
		  {
			"gNbId": {
			  "bitLength": 24,
			  "gNBValue": ""
			},
			"plmnId": {
				"mnc": "",
				"mcc": ""
			},
			"n3IwfId": "",
			"ngeNbId": ""
		  }
		],
		"anTypeList":[

		],
		"n2InformationClass": "PWS",
		"n2NotifyCallbackUri": "127.0.0.1:8080/notify",
		"nfId": "",
		"supportedFeatures": ""
	  }`)

	req, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonString))
    req.Header.Set("X-Custom-Header", "myvalue")
    req.Header.Set("Content-Type", "application/json")
    client := &http.Client{}
    response, err := client.Do(req)
	if err != nil {
		fmt.Printf("Error sending request: %s\n", err)
		return
	}
	defer response.Body.Close()

	body, err := ioutil.ReadAll(response.Body)
	if err != nil {
		fmt.Printf("Error reading response: %s\n", err)
		return
	}
	fmt.Println(string(body))
}

```

:::
:::info
The JSON is hardcoded in the code and will be sent as a body in HTTP Request to Non UE N2 subscribe info in AMF.
If we see the OpenAPI, the schema of the JSON is described in the models.NonUeN2InfoSubscriptionCreateData, that can be seen at [3GPP SPEC TS 29 518]
![](https://hackmd.io/_uploads/rJTSmi-82.png)
:::
:::info
The complete details of the NonUeN2InfoSubscriptionCreateData:
* globalRanNodeList: List of RAN Nodes, 
    * Identified by gnbID, the global Node base station ID
* anTypeList: List of access network type that needs to be subscribed, the access network type is enumeration of 'AccessType__3_GPP_ACCESS' or 'AccessType_NON_3_GPP_ACCESS'
* n2InformationClass: enumeration to choose class of the N2 message, for example PWS, SM, RAN
* n2NotifyCallbackUri: the callback URI on which the N2 information shall be notified
* nfId: the identifier of network function instance id


The mandatory parameters is the n2InformationClass and n2NotifyCallbackUri the rest is optional
:::

* Modify NonUeN2MessagesSubscriptionsCollection to print request message
```
cd NFs/amf/internal/sbi/communication/
nano api_non_uen2_messages_subscriptions_collection_document.go
make amf
```
![](https://hackmd.io/_uploads/BJqLIoWI3.png)

![](https://hackmd.io/_uploads/rJjmOj-U3.png)

![](https://hackmd.io/_uploads/BJM8_jZI2.png)


* Run Free5GC, open a new terminal and send the request
```
./run.sh
```
![](https://hackmd.io/_uploads/Hyau_j-8h.png)

```
go run NonUeN2InfoSubscribe.go
```
![](https://hackmd.io/_uploads/rJY1toWI3.png)

![](https://hackmd.io/_uploads/H1IXFjbIn.png)
 
:::success
The message is successfully received in the AMF
:::

### Non UE N2 Info UnSubscribe:
To use Non UE N2 Info UnSubscribe in AMF we will create a go program to acts as HTTP client and send a HTTP Request to use Non UE N2 info subscribe in AMF.
* Create the code
```
nano NonUeN2InfoUnsubscribe.go
```
![](https://hackmd.io/_uploads/BJL6Fjb8h.png)

![](https://hackmd.io/_uploads/Hkg-5ibI2.png)

:::spoiler The code:
```
package main

import (
	"context"
	"fmt"
	"io/ioutil"
	"github.com/free5gc/openapi/Namf_Communication"
)

func unsubscribe(id string) {
	// Specify the URL you want to send the request to
	namfConfiguration := Namf_Communication.NewConfiguration()
	namfConfiguration.SetBasePath("http://127.0.0.18:8000")
	apiClient := Namf_Communication.NewAPIClient(namfConfiguration)
	res, err := apiClient.NonUEN2MessageNotificationIndividualSubscriptionDocumentApi.NonUeN2InfoUnSubscribe(context.TODO(), id)
	
	body, err := ioutil.ReadAll(res.Body)
	if err != nil {
		fmt.Printf("Error reading response: %s\n", err)
		return
	}
	fmt.Println(string(body))

}

```

:::
:::info
If we see the OpenAPI, this API only needs n2NotifySubscriptionId which is in the request URI params, so we create the code to use flag `id` when running the program to specify the id, and also this API uses HTTP DELETE method, instead of POST method like the others
:::
![](https://hackmd.io/_uploads/SJJ1ij-L3.png)

:::info
N2 Notify Subscription id is the Id created by the AMF for the subscription to notify a non UE related N2 information. 
:::

* Modify NonUeN2MessagesNotificationIndividualSubscription to print request message
```
cd NFs/amf/internal/sbi/communication/
nano api_non_uen2_messages_notification_individual_subscription_document.go
make amf
```
![](https://hackmd.io/_uploads/BJp00obIn.png)

![](https://hackmd.io/_uploads/rJG-1nWL3.png)


* Run Free5GC, open a new terminal and send the request
```
./run.sh
```
![](https://hackmd.io/_uploads/BJCXk3-L2.png)


```
go run NonUeN2InfoSubscribe.go -id 1234
```
![](https://hackmd.io/_uploads/S1lC12-I2.png)

![](https://hackmd.io/_uploads/r1hZe2ZL2.png)

:::info
In the code, we use flag to input the id so when running the program, we must specify the id by using `-id` flag
:::
:::success
The message is successfully received in the AMF
:::

### Server
The CBCF should also work as a server when receiving emergency broadcast request from CBE, so we will try to create a Go Program for CBCF to act as HTTP Server
```
nano server.go
```
![](https://hackmd.io/_uploads/H1dRokPIn.png)

![](https://hackmd.io/_uploads/rybQ3ywLh.png)

:::spoiler The code:
```go=
package main

import (
	"log"
	"os/exec"
	"net/http"
	"io/ioutil"
	"fmt"
)

func main() {
	http.HandleFunc("/", handleRequest)
	http.HandleFunc("/notify", handleNotify)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
	cmd := exec.Command("go", "run", "NonUeN2InfoSubscribe.go")
	output, err := cmd.Output()
	
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("Subscribe output:\n%s", output)
	w.Write(output)

	cmd = exec.Command("go", "run", "NonUEN2MessageTransferRequest.go")
	output, err = cmd.Output()
	
	if err != nil {
		log.Fatal(err)
	}
	log.Printf("Message Transfer output:\n%s", output)
	w.Write(output)
}

func handleNotify(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}
	body, err := ioutil.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Failed to read request body", http.StatusInternalServerError)
		return
	}
	fmt.Println(string(body))
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Received the request body"))
}
```

:::
:::info
The code is to run CBCF as HTTP Server to implement the [warning message delivery](https://hackmd.io/@joshevan/3GPP-TS-23041#912-Warning-Message-Delivery-in-NG-RAN).

The diagram:
![](https://hackmd.io/_uploads/SJmGyBED3.png)

The design of the program:
* The available endpoint is `/` with POST Method, it is used to simulate the message sent from CBE, the message from the CBE will be received from the request body
* After that, it will call the `subscribe()`, it is used to call the NonUeN2Subscribe to subscribe to the AMF like in the procedure
* After that, it will call the `transfer(m)`, it is used to call the NonUeN2InfoSubscribe to send the message to AMF, m is the request body of the message
* The server program also has `/notify` endpoint, it is used for the NonUeN2InfoNotify, so the AMF will notify the CBCF. The notify will be sent via that endpoint and that endpoint is specified in the NonUeN2InfoSubscribe parameter as n2NotifyCallbackUri
* The design of the program will run the CBCF for the warning message delivery procedure.
* Before running the program we must build the program using `go build` and then run it using `./CBCF.exe` 
:::

* Run the free5gc
```
./run.sh
```
![](https://hackmd.io/_uploads/H1BX0kw82.png)

* Run the server.go
```
cd cbcftest
go run server.go
```
![](https://hackmd.io/_uploads/Hya401wU3.png)

* Access the CBCF endpoint using curl
```
curl 127.0.0.1:8080/
```
![](https://hackmd.io/_uploads/ry_9AyvU2.png)

* The output in server and free5gc

![](https://hackmd.io/_uploads/HklnA1vL2.png)

![](https://hackmd.io/_uploads/SJ1aA1DL3.png)

:::success
The message is successfully handled by the CBCF server and is forwarded into the free5gc. For the next part, we will use a program to simulate the CBE then modify the CBCF server to decode the message, then forward it as parameters to NonUeN2Message, so it will fully functional as CBCF
:::

* Modify the program to receive parameters from request body
We will modify the program to receive parameters from request body, like the tac, mnc, mcc, and the ratSelector for the write replace warning request message
:::spoiler The code:
```go=
package main

import (
	"log"
	"net/http"
	"io/ioutil"
	"flag"
	"fmt"
)

func main() {
	http.HandleFunc("/", handleRequest)
	http.HandleFunc("/notify", handleNotify)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}
	if err := r.ParseForm(); err != nil {
		http.Error(w, "Error parsing form data", http.StatusBadRequest)
		return
	}
	ratSelector := r.Form.Get("ratSelector")
	tac := r.Form.Get("tac")
	mnc := r.Form.Get("mnc")
	mcc := r.Form.Get("mcc")
	n2Info := r.Form.Get("n2Info")
	flag.Parse()
	m := make(map[string]string)
	m["ratSelector"] = ratSelector
	m["tac"] = tac
	m["mnc"] = mnc
	m["mcc"] = mcc
	m["n2Info"] = n2Info
	subscribe()
	transfer(m)
}

func handleNotify(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}
	body, err := ioutil.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Failed to read request body", http.StatusInternalServerError)
		return
	}
	fmt.Println(string(body))
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Received the request body"))
}
```
:::

* Build the program
```
go build 
```
![](https://hackmd.io/_uploads/rkCqycjL3.png)

* Run free5gc
![](https://hackmd.io/_uploads/BkPAy9i8h.png)

* Run the server
```
./CBCF
```
![](https://hackmd.io/_uploads/Sy2MeciLn.png)

* Access the endpoint using CURL with data
```
curl -X POST -d "ratSelector=NR&tac=tac&mnc=30&mcc=20" http://localhost:8080/
```
![](https://hackmd.io/_uploads/BybxZcjL3.png)

* Output in AMF (free5gc)

![](https://hackmd.io/_uploads/SJXVb9jU2.png)

* Output in Wireshark:

![](https://hackmd.io/_uploads/HJngF0vO3.png)

:::success
The message and the parameters is succesffully sent
:::
:::info
There is several packages captured in wireshark the complete explanation:
* First Package: The cURL to the CBCF server
* Second Package: The CBCF send the NonUeN2MessageSubcscribe to the AMF (Free5GC)
* Third Package: The AMF (Free5GC) responds with HTTP 200 OK
* Fourth Package: The CBCF sends the NonUeN2MessageTransfer to the AMF (Free5GC)
* Fifth Package: The AMF (Free5GC) temporary redirect it with status code 307
* Sixth Package: The CBCF sends again the NonUeN2MessageTransfer to the AMF (Free5GC)
* Seventh Package: The AMF (Free5GC) responds with HTTP 200 OK
* Eighth Package: The CBCF responds with HTTP 200 OK to the client (cURL)
:::
:::info
In this version, we use `Go Build` to build the package because it has already combined few programs, after building the package, there will be an executable file to run
:::

#### Add /unsubscribe endpoint in the server
In this version, the package is already compiled as one function, so we cannot use the program individually. To use the unsubscribe function, we have to modify the server so that it has and endpoint that will call the unsubscribe function.
:::spoiler The code:
```
func handleUnsubscribe(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodDelete {
		http.Error(w, r.Method, http.StatusMethodNotAllowed)
		return
	}
	id := r.URL.Path[len("/unsubscribe/"):]
	unsubscribe(id)
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Received the request body"))
}
```
![](https://hackmd.io/_uploads/BkaRrl2O2.png)
:::
:::info
The details of the code:
* It will add an endpoint of `/unsubscribe/:id`, with the id is the params that will be unsubscribed
* After that it will call the unsubscribe function
* The unsubscribe function is also modified a little bit by so that it can take the id as a parameter 
:::
Testing the endpoint using cURL:
```
curl -X DELETE 127.0.0.1:8080/unsubscribe/1
```
![](https://hackmd.io/_uploads/rJzp8lh_2.png)
:::success
The endpoint is able to used
:::

### Add Database functionality to CBCF
We will try to add database functionality to CBCF, so that the CBCF will be able to save the message into database. The database that will be used is MongoDB
#### Database Functionality in NonUeN2TransferMessage
* Enter the MongoDB shell:
```
mongo
```
![](https://hackmd.io/_uploads/B16FZr4w3.png)

* Create new collection in MongoDB
```
db.createCollection("CBCF")
```
![](https://hackmd.io/_uploads/BksSzHNvn.png)

* Exit the MongoDB and edit the NonUeN2Transfer Message
```
exit
cd cbcftest/
nano NonUEN2MessageTransferRequest.go
```
![](https://hackmd.io/_uploads/SygpGBNP3.png)

![](https://hackmd.io/_uploads/SJLCfHEw3.png)

* Add the insertToDatabase function
![](https://hackmd.io/_uploads/Hk0z7z_P3.png)


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
	collection := client.Database("local").Collection("cbcf")
	insertResult, err := collection.InsertOne(context.TODO(), message)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Inserted document ID: %v\n", insertResult.InsertedID)
	sort := options.FindOne().SetSort(bson.D{{"_id", -1}})
	var result models.NonUeN2MessageTransferRequest
	err = collection.FindOne(context.TODO(), bson.D{}, sort).Decode(&result)
	var b []byte
	b, err = json.Marshal(result) 
	fmt.Println(string(b))

}
```
:::

* Build the code
```
go build
```
![](https://hackmd.io/_uploads/Hy2jXS4P3.png)

* Try the function:
    * Run the CBCF
    ```
    ./CBCF
    ```
    * Open a new terminal and run free5gc
    ```
    cd free5gc/
    ./run.sh
    ```
    ![](https://hackmd.io/_uploads/Bk9BVSNDh.png)
    
    * Open a new terminal again and send the message using cURL
    ```
    curl -X POST -d "ratSelector=NR&tac=tac&mnc=30&mcc=20" http://localhost:8080/
    ```
    ![](https://hackmd.io/_uploads/SJGMIBVwn.png)

    * See the result in CBCF and free5gc
    
    ![](https://hackmd.io/_uploads/S1T-4MuDn.png)
    
    ![](https://hackmd.io/_uploads/rJzDIr4D2.png)

    * Enter the MongoDB shell again to see the message
    ```
    mongo
    db.cbcf.find().pretty()
    ```
    ![](https://hackmd.io/_uploads/BJHN4Gdwn.png)

:::success
The data is succesfully inserted into the MongoDB database
:::

### CBE Simulator
In real cases, the CBCF should receive data from CBE (Cell Broadcast Entity), we will try to create a CBE simulator, so that we will use that to send the data, we don't use the cURL anymore.
The CBE will be written in `python`, below is the code:
``` python
import requests
import argparse

parser = argparse.ArgumentParser(description='The parameter for the CBS message')
parser.add_argument('-id', '--messageId', type=int, help='The message ID', required = True)
parser.add_argument('-rs', '--ratSelector', type=str, help='Enumeration to choose between E-UTRA (4G) and NG (5G), the default is 5G', choices=['E-UTRA', 'NR'],  default='NR')
parser.add_argument('-n2', '--n2InformationClass', type=str, help='Enumeration to choose between class of the N2 message, it can be PWS, SM, RAN, NRPPa, PWS-BCAL, the default is PWS', choices=['PWS', 'SM', 'RAN', 'NRPPa', 'PWS-BCAL'],  default='PWS')
parser.add_argument('-t', '--tac', type=str, help='Parameter 1', default = '')
parser.add_argument('-mn', '--mnc', type=int, help='Mobile Network Code')
parser.add_argument('-mc', '--mcc', type=int, help='Mobile Country Code')

args = parser.parse_args()

# Prepare the parameters
data = {
    'id': args.messageId,
    'ratSelector': args.ratSelector,
    'n2Information': args.n2InformationClass,
    'tac': args.tac,
    'mnc': args.mnc,
    'mcc': args.mcc
}

# Send the HTTP request
response = requests.post('http://127.0.0.1:8080/', data=data)

# Check the response status code
if response.status_code == requests.codes.ok:
    print('Request was successful.')
    print('Response:', response.text)
else:
    print('Request failed with status code:', response.status_code)

```

:::info
This python program is used to simulate the CBE, it will send the data into the CBCF through the '/'. To specify the data, we used the flag when running the program. Command to check the flag:
```
python3 CBE.py -h
```
![](https://hackmd.io/_uploads/S1KmURvu3.png)

In this program we use some arguments for the flag:
* `-id --messageId`: The message id 
* `-rs --ratSelector`: Enumeration to choose between E-UTRA(4G) and NR (5G), the default value is 5G
* `-n2 --n2InformationClass`: Enumeration to choose between class of the N2 message, it can be PWS, SM, RAN, NRPpa, PWS-BCAL, the default value is PWS
* `-t --tac`: Tracking Area Code
* `-mn --mnc`: Mobile Network Code
* `-mc --mcc`: Mobile Country Code
:::
After that, we will run the program by specifying the arguments:
```
python3 CBE.py --ratSelector=NR --tac=tac --mnc=30 --mcc=20 --messageId=1 --n2InformationClass=PWS
```
![](https://hackmd.io/_uploads/B1UvwRPd2.png)

The message was successfully sent, the output in CBCF:
![](https://hackmd.io/_uploads/HkmcP0vd2.png)

The output in Free5GC
![](https://hackmd.io/_uploads/B1w2DRwdh.png)

Output in wireshark:

![](https://hackmd.io/_uploads/HJngF0vO3.png)

:::success
The message is successfully received in the CBCF with the arguments provided, and then the message is also succesffully received in the Free5GC (AMF)
:::
:::info
The output in wireshark is the same before, the only difference is now it uses python program, instead of cURL only
:::

### Trying to use the openAPI
In this part we will try to use the openAPI repository in Free5gc, the repository contains several functions, including the function for the NonUeN2TransferMessage, NonUeN2TransferSubscribe, and the NonUeN2TransferUnSubscribe. The Function is it create a standarized HTTP Request to the AMF. The details of the API used can be seen [here](https://github.com/free5gc/openapi/tree/main/Namf_Communication)

#### NonUeN2TransferMessage
First we will try to implement the openAPI for the NonUeN2TransferMessage. 
:::spoiler The added code:
``` go
namfConfiguration := Namf_Communication.NewConfiguration()
namfConfiguration.SetBasePath("http://127.0.0.18:8000")
apiClient := Namf_Communication.NewAPIClient(namfConfiguration)
rep, res, err := apiClient.NonUEN2MessagesCollectionDocumentApi.NonUeN2MessageTransfer(context.TODO(), message)
```
:::
![](https://hackmd.io/_uploads/r1vlbe3_2.png)
:::info
The details of the code:
* First we create a new configuration of Namf Communication, and then we set the base path, which is the IP Address of the AMF Namf Communication (127.0.0.18:8000)
* After that we create a new API Client from that configuration
* Last, we call the function by calling the NonUeN2MessageTransfer function from the API Client
:::

#### NonUeN2InfoSubscribe
We will also try to implement the openAPI for the NonUeN2InfoSubscribe
:::spoiler The added code:
``` go
namfConfiguration := Namf_Communication.NewConfiguration()
namfConfiguration.SetBasePath("http://127.0.0.18:8000")
apiClient := Namf_Communication.NewAPIClient(namfConfiguration)
rep, res, err := apiClient.NonUEN2MessagesSubscriptionsCollectionDocumentApi.NonUeN2InfoSubscribe(context.TODO(), subscribe)
```
:::
![](https://hackmd.io/_uploads/rJ5zze2_2.png)

:::info
The details of the code:
* Same like before, first we create a new configuration of Namf Communication, and then we set the base path, which is the IP Address of the AMF Namf Communication (127.0.0.18:8000)
* After that we create a new API Client from that configuration
* Last, we call the function by calling the NonUeN2InfoSubscribe function from the API Client
:::


#### NonUeN2InfoUnSubscribe
We will also try to implement the openAPI for the NonUeN2InfoUnSubscribe
:::spoiler The added code:
``` go
namfConfiguration := Namf_Communication.NewConfiguration()
namfConfiguration.SetBasePath("http://127.0.0.18:8000")
apiClient := Namf_Communication.NewAPIClient(namfConfiguration)
rep, res, err := apiClient.NonUEN2MessagesSubscriptionsCollectionDocumentApi.NonUeN2InfoSubscribe(context.TODO(), subscribe)
```
:::
![](https://hackmd.io/_uploads/SycvXgh_3.png)



:::info
The details of the code:
* Same like before, first we create a new configuration of Namf Communication, and then we set the base path, which is the IP Address of the AMF Namf Communication (127.0.0.18:8000)
* After that we create a new API Client from that configuration
* Last, we call the function by calling the NonUeN2InfoUnSubscribe function from the API Client
:::

#### Testing of the API
After using the openAPI, we will try to run it again to see if it works successfully

* Running the Free5GC
```
cd free5gc/
./run.sh
```
![](https://hackmd.io/_uploads/S1RTQlhOh.png)


* Running the CBCF
```
cd cbcftest/
go build
./CBCF
```
![](https://hackmd.io/_uploads/ry6qXl2d3.png)

* Running the CBE
```
cd cbcftest/
python3 CBE.py --ratSelector=NR --tac=tac --mnc=30 --mcc=20 --messageId=1 --n2InformationClass=PWS
```
![](https://hackmd.io/_uploads/B19zEg3u3.png)

* Output in Free5gc

![](https://hackmd.io/_uploads/rJytVx3_n.png)

* Output in CBCF

![](https://hackmd.io/_uploads/rJ294gh_n.png)

:::success
The message is succesfully received in the Free5gc, it means that the openAPI is successfully used. Using openAPI helps to guarantee the standard of the HTTP Request message, so that it can be handled in the AMF
:::

List of supported features:
* CBCF can receive data from CBE, but still not in the correct format, like CAP
* CBCF can subscribe to the AMF (NonUeInfoSubscribe)
* CBCF can forward data to the AMF (NonUeN2Message)
* CBCF can delete the subscribe data by using id (mongoDB id)
* The AMF only receive the subscribe data and insert it into the database, but does not have the handle for it
* The AMF receive NonUeN2Message data and transfer it into the gNB by building Write Replace Warning Request message

### Modify the CBCF to insert the data from the CBE and forward the data into the AMF

In the previous version, the write replace warning request message is still emtpy, we will try to input the data from the CBE and forward it to be used by AMF to build Write Replace Warning Request Package. The code in the server:
``` go=
id := r.Form.Get("id")
tac := r.Form.Get("tac")
mnc := r.Form.Get("mnc")
mcc := r.Form.Get("mcc")
n2Information := r.Form.Get("n2Information")
repetitionPeriod := r.Form.Get("repetitionPeriod")
numberOfBroadcastsRequested := r.Form.Get("numberOfBroadcastsRequested")
warningMessageContents := r.Form.Get("warningMessageContents")
m := make(map[string]string)
m["ratSelector"] = ratSelector
m["id"] = id
m["tac"] = tac
m["mnc"] = mnc
m["mcc"] = mcc
m["n2Information"] = n2Information
m["repetitionPeriod"] = repetitionPeriod
m["numberOfBroadcastsRequested"] = numberOfBroadcastsRequested
m["warningMessageContents"] = warningMessageContents
```
The code in NonUEN2MessageTransfer
``` go=
BinaryDataN2InformationKeyValue := make(map[string]interface{})
json.Unmarshal([]byte(BinaryDataN2informationString), &BinaryDataN2InformationKeyValue)
BinaryDataN2InformationKeyValue["messageIdentifier"] = "1112"
BinaryDataN2InformationKeyValue["serialNumber"] = "5940"
BinaryDataN2InformationKeyValue["repetitionPeriod"] = m["repetitionPeriod"]
BinaryDataN2InformationKeyValue["numberOfBroadcastsRequested"] = m["numberOfBroadcastsRequested"]
BinaryDataN2InformationKeyValue["dataCodingScheme"] = "48"
BinaryDataN2InformationKeyValue["warningMessageContents"] = time.Now().Format("2006-01-02 15:04:05") + ":" + m["warningMessageContents"]
```

In that code, the data from the CBE will be converted into map of string, the data will then be used by the NonUeN2MessageTransfer to be put into the request data and then send to the AMF to be used by AMF to build Write Replace Warning Request. Next the CBCF will be modified more to match the CAP (Common Alerting Protocol), so that it matches the real cases when it receive data from CBE

### Modify the CBE and CBCF to match the CAP-TWP
CAP-TWP (Common Alerting Protocol Taiwan) is a protocol for the CBE to send the data to CBCF, it defines several attributes and its value in XML format, the complete reference can be seen [here](https://alerts.ncdr.nat.gov.tw/capinfo.aspx). The CBE must be modified to be able to send data through HTTP in XML format, the CBCF must also be modifier to be able to receive XML data and match the format to build a Write Replace Warning Request to the AMF.
The code in CBE:
```python=
import requests

def send_xml_data(url, xml_data):
    headers = {
        "Content-Type": "application/xml ;charset=utf-8",
    }
    
    try:
        response = requests.post(url, data=xml_data, headers=headers)
        response.raise_for_status()
        return response.content
    except requests.exceptions.RequestException as e:
        print(f"Error: {e}")
        return None

# Example XML data
xml_data = """
<?xml version="1.0" encoding="UTF-8"?> 
<alert xmlns="urn:oasis:names:tc:emergency:cap:1.1">
    <identifier>CWB-EQ112214</identifier> 
    <sender>cwb@scman.cwb.gov.tw</sender> 
    <sent>2023-07-27T00:08:00+08:00</sent> 
    <status>Actual</status> 
    <msgType>Alert</msgType>
    <source>CWB</source>
    <scope>Public</scope> 
    <info> 
        <language>zh-TW</language>
        <category>Met</category>
        <event>地震</event>
        <responseType>Shelter</responseType>
        <urgency>Immediate</urgency>
        <severity>Severe</severity>
        <certainty>Observed</certainty>
        <effective>2023-07-27T08:00:00+08:00</effective>
        <expires>2023-07-27T08:08:00+08:00</expires> 
        <senderName>中央氣象局</senderName> 
        <headline>地震報告</headline> 
        <description> 07/27-18:11 花蓮縣秀林鄉發生規模 5.3 有感地震，最大震度花蓮縣太魯閣、宜蘭縣南山、南投縣合歡山、臺中市德基 4 級。</description>
        <contact>123456</contact>  
        <area> 
            <areaDesc>最大震度 3 級地區</areaDesc> 
            <geoCode>10002</geoCode> 
        </area> 
        </info> 
    </alert>
"""

# Example URL to which you want to send the XML data
url = 'http://127.0.0.1:8080'
encoded_xml_data = xml_data.encode('utf-8')

# Sending the XML data and getting the response
response_content = send_xml_data(url, encoded_xml_data)
if response_content:
    print("Response:")
    print(response_content)
else:
    print("Failed to send XML data.")

```

In the code, the XML format is written in String and then the header is set to application/xml, then the data is encoded in utf-8 to be able to send Taiwanese language message. After that the data will be send into the CBCF.

The code in CBCF:
```go=
package main

import (
	"encoding/xml"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"strconv"
	"time"
)

// Define the Go struct representing the XML data
type Alert struct {
	XMLName    xml.Name  `xml:"urn:oasis:names:tc:emergency:cap:1.1 alert"`
	Identifier string    `xml:"identifier"`
	Sender     string    `xml:"sender"`
	Sent       time.Time `xml:"sent"`
	Status     string    `xml:"status"`
	MsgType    string    `xml:"msgType"`
	Scope      string    `xml:"scope"`
	Source     string    `xml:"source"`
	Info       Info      `xml:"info"`
}

type Info struct {
	Language     string    `xml:"language"`
	Category     string    `xml:"category"`
	Event        string    `xml:"event"`
	ResponseType string    `xml:"responseType"`
	Urgency      string    `xml:"urgency"`
	Severity     string    `xml:"severity"`
	Certainty    string    `xml:"certainty"`
	Expires      time.Time `xml:"expires"`
	SenderName   string    `xml:"senderName"`
	Headline     string    `xml:"headline"`
	Description  string    `xml:"description"`
	Contact      string    `xml:"contact"`
	Area         Area      `xml:"area"`
}

type Area struct {
	AreaDesc string `xml:"areaDesc"`
	Polygon  string `xml:"polygon"`
	GeoCode  string `xml:"geocode"`
}

func main() {
	http.HandleFunc("/", handleRequest)
	http.HandleFunc("/notify", handleNotify)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	xmlData, err := ioutil.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Error parsing body data", http.StatusBadRequest)
		return
	}
	var alertData Alert
	if err := xml.Unmarshal(xmlData, &alertData); err != nil {
		fmt.Println(err)
		http.Error(w, "Error parsing XML data", http.StatusBadRequest)
		return
	}
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("XML data received successfully"))
	data := make(map[string]string)
	serialNumberInteger, err := strconv.Atoi(alertData.Identifier[len(alertData.Identifier)-3:])
	serialNumber := int64(serialNumberInteger)
	serialNumberBits := strconv.FormatInt(int64(serialNumber), 2)
	for len(serialNumberBits) < 10 {
		serialNumberBits = "0" + serialNumberBits
	}
	serialNumberBits = "11" + serialNumberBits + "0000"
	serialNumber, err = strconv.ParseInt(serialNumberBits, 2, 64)
	data["serialNumber"] = fmt.Sprintf("%x", serialNumber)
	fmt.Println(data["serialNumber"])
	data["messageType"] = alertData.MsgType
	if alertData.Info.Language == "en-US" {
		data["dataCodingScheme"] = "01"
	}
	if alertData.Info.Language == "zh-TW" {
		data["dataCodingScheme"] = "48"
	}
	switch {
	case alertData.Info.Severity == "Extreme" && alertData.Info.Urgency == "Immediate" && alertData.Info.Certainty == "Observed":
		data["messageIdentifier"] = "1113"
	case alertData.Info.Severity == "Extreme" && alertData.Info.Urgency == "Immediate" && alertData.Info.Certainty == "Likely":
		data["messageIdentifier"] = "1114"
	case alertData.Info.Severity == "Extreme" && alertData.Info.Urgency == "Expected" && alertData.Info.Certainty == "Observed":
		data["messageIdentifier"] = "1115"
	case alertData.Info.Severity == "Extreme" && alertData.Info.Urgency == "Expected" && alertData.Info.Certainty == "Likely":
		data["messageIdentifier"] = "1116"
	case alertData.Info.Severity == "Severe" && alertData.Info.Urgency == "Immediate" && alertData.Info.Certainty == "Observed":
		data["messageIdentifier"] = "1117"
	case alertData.Info.Severity == "Severe" && alertData.Info.Urgency == "Immediate" && alertData.Info.Certainty == "Likely":
		data["messageIdentifier"] = "1118"
	case alertData.Info.Severity == "Severe" && alertData.Info.Urgency == "Expected" && alertData.Info.Certainty == "Observed":
		data["messageIdentifier"] = "1119"
	case alertData.Info.Severity == "Severe" && alertData.Info.Urgency == "Expected" && alertData.Info.Certainty == "Likely":
		data["messageIdentifier"] = "111A"
	default:
		data["messageIdentifier"] = "1112"
	}
	data["warningMessageContents"] = alertData.Sent.Format("2006-01-02 15:04:05") + alertData.Info.Headline + "\n" + alertData.Info.Description + "\n" + alertData.Info.Area.AreaDesc
	data["tac"] = alertData.Info.Area.GeoCode
	subscribe()
	transfer(data)
}

func handleNotify(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}
	body, err := ioutil.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Failed to read request body", http.StatusInternalServerError)
		return
	}
	fmt.Println(string(body))
	w.WriteHeader(http.StatusOK)
	w.Write([]byte("Received the request body"))
}

```
In the code, first we define the struct for the CAP format, so that it match the message, after that the data will be unmarshalled into the struct, so we have an object with all of the attributes. After that we create a map of string for the data to be converted and be used to be sent into the AMF. The conversion from the CAP-TWP format to Write Replace Warning Request is according to this table:

![](https://hackmd.io/_uploads/Hkj50Skjn.jpg)

We convert it to match it, like we convert the identifier to serial number, we use severity, urgency, and certainty to determine the message identifier, we also use time sent, headline, description, and area description to put into the Warning Message Content.

After that we will try to run it:
* Running the Free5GC:
```
./run.sh
```
![](https://hackmd.io/_uploads/HyC1x8Ji3.png)

* Running the CBCF:
```
./CBCF
```
![](https://hackmd.io/_uploads/BJWbxL1o3.png)


* Running the UERANSIM:
```
build/nr-gnb -c config/free5gc-gnb.yaml
```
![](https://hackmd.io/_uploads/SywmlLJj3.png)

* Running the tcpdump:
```
date +'%Y-%m-%d_%H.%M.%S' | xargs -I {} bash -c "sudo tcpdump -i ens18 -w ./{}.pcap"
```
![](https://hackmd.io/_uploads/SkeDgI1s3.png)

* Running the CBE Simulator:
```
python3 CBE.py
```
![](https://hackmd.io/_uploads/Sym9eUys3.png)

The output in wireshark:
![](https://hackmd.io/_uploads/Hk4fWU1j3.png)

![](https://hackmd.io/_uploads/BkPVZ8Jo2.png)

The message is succefully received and is the same with the format in CBE, so it works successfully. In this code, the data in the CBE is still hardcoded, in the next part we will use input, like before, so that the data can be changed dynamically.

The output in UE:
![](https://hackmd.io/_uploads/rJgcKQ0s2.jpg)

### Modify the CBE and CBCF for Serial Number
In the PWS, if the UE receive the same message identifier, serial number, and update number it will discard the message so it does not show in the UE. We will create a program to take serial number from input in the CBE, and also in the CBCF, so if the message from CBE has the same message identifier and serial number, it will increment the update number.

The added code in CBE:
```go=
parser = argparse.ArgumentParser(description='The parameter for the CBS message')
parser.add_argument('-sn', '--serialNumber', type=int, help='The message ID', required = True)
args = parser.parse_args()
root = ET.fromstring(xml_data)
serialNumber_element = root.find('.//{urn:oasis:names:tc:emergency:cap:1.1}identifier')
if serialNumber_element is not None:
            serialNumber_element.text = serialNumber_element.text[:-3] + f"{args.serialNumber:03d}"
modified_xml_string = ET.tostring(root, encoding='utf-8', method='xml')
```

In that code, it will get the argument from user input by `-sn` or `--serialNumber` for the serial number. After getting the input, it will search the XML for the identifier and add the serial number to the identifier

The added code in CBCF (NonUeN2MessageTransferRequest):
``` go =
func countMessageFromDatabase(messageIdentifier string, serialNumber string) int64 {
	messageIdentifierint, err := strconv.Atoi(messageIdentifier)
	serialNumberint, err := strconv.Atoi(serialNumber)
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")
	client, err := mongo.Connect(context.Background(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}
	collection := client.Database("local").Collection("cbcf")
	var result int64
	condition := bson.M{"jsondata.n2information.pwsinfo.messageidentifier": messageIdentifierint, "jsondata.n2information.pwsinfo.serialnumber": serialNumberint}
	result, err = collection.CountDocuments(context.Background(), condition)
	if err != nil {
		log.Fatal(err)
	}
	return result
}

countNumber := countMessageFromDatabase(data["messageIdentifier"], data["serialNumber"])
	var serialNumber int64
	if countNumber >= 0 {
		serialNumberInteger, err := strconv.Atoi(data["serialNumber"])
		if err != nil {
			fmt.Println(err)
		}
		fmt.Println(data["serialNumber"])
		serialNumberBits := "01" + "00" + fmt.Sprintf("%08b", serialNumberInteger) + fmt.Sprintf("%04b", countNumber)
		fmt.Println(serialNumberBits)
		serialNumber, err = strconv.ParseInt(serialNumberBits, 2, 64)
	}
	serialNumber64, err := strconv.Atoi(data["serialNumber"])
	message.JsonData.N2Information.PwsInfo.SerialNumber = int32(serialNumber64)
	messageIdentifier, err := strconv.Atoi(data["messageIdentifier"])
	message.JsonData.N2Information.PwsInfo.MessageIdentifier = int32(messageIdentifier)
	BinaryDataN2InformationKeyValue["serialNumber"] = fmt.Sprintf("%x", serialNumber)
```

In that code I added the function of countMessageFromDatabase, it will search for the number of message with same message identifier and serial number, it will affect the update number. After that, the function is called to create the string of hex for serial number. 

Test result:
![](https://hackmd.io/_uploads/S12khX0ih.jpg)

![](https://hackmd.io/_uploads/S1hyhm0sh.jpg)

![](https://hackmd.io/_uploads/SJiJ2mCsn.jpg)

The test is it success to receive the same message for several times, it means that the serial number has been configured correctly
