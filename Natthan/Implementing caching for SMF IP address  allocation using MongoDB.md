In this implementation we are going to implement DHCP Concept Function the the 5G Core SMF (Free5GC). 

# Implementation
In this implementation we are going to store the DHCP information inside the MongoDB. The data stored in the MongoDB will be containing 

## Key 
* UE SUPI
* UPF Selection Parameters
## Value
* IP address
* Expiration Date

We use the UE SUPI and the UPF Selection Parameters as the key that will give the required parameters to give the IP address for the UE and the expiration time for the IP address lease. 

# Diagram
## Flowchart
![](https://hackmd.io/_uploads/ByccE37Y2.png)

## Sequence Diagram
![](https://hackmd.io/_uploads/rJbyHn7K3.png)


# Source Code
In the Final Implementation, we make modification in several part of the free5GC SMF. 

* pdu_session.go
* user_plane_information.go
* dhcpservice.go

Here are the new changes made

## pdu_session.go
``` go=
var selectedUPF *smf_context.UPNode
	var ip net.IP
	selectedUPFName := ""
	if smf_context.SMF_Self().ULCLSupport && smf_context.CheckUEHasPreConfig(createData.Supi) {
		groupName := smf_context.GetULCLGroupNameFromSUPI(createData.Supi)
		defaultPathPool := smf_context.GetUEDefaultPathPool(groupName)
		if defaultPathPool != nil {
			selectedUPFName, ip = defaultPathPool.SelectUPFAndAllocUEIPForULCL(
				smf_context.GetUserPlaneInformation(), upfSelectionParams)
			selectedUPF = smf_context.GetUserPlaneInformation().UPFs[selectedUPFName]
		}
	} else {
		selectedUPF, ip = smf_context.GetUserPlaneInformation().SelectUPFAndAllocUEIP(upfSelectionParams, createData.Supi)
		smContext.PDUAddress = ip
		fmt.Println(smContext.PDUAddress)
		logger.PduSessLog.Infof("UE[%s] PDUSessionID[%d] IP[%s]",
			smContext.Supi, smContext.PDUSessionID, smContext.PDUAddress.String())
	}
```

In the source code above we will modify the SelectUPFAndAllocUEIP function by modifying its parameter and give the function an additional parameter which is SUPI. 

## user_plane_information.go
```go=
func (upi *UserPlaneInformation) SelectUPFAndAllocUEIP(selection *UPFSelectionParams, supi string) (*UPNode, net.IP) {
	DHCPKey := dhcpKey{
		Selection: selection,
		Supi:      supi,
	}

	source, err := upi.selectUPPathSource()
	if err != nil {
		return nil, nil
	}

	UPFList := upi.selectAnchorUPF(source, selection)
	if len(UPFList) == 0 {
		logger.CtxLog.Warnf("Can't find UPF with DNN[%s] S-NSSAI[sst: %d sd: %s] DNAI[%s]\n", selection.Dnn,
			selection.SNssai.Sst, selection.SNssai.Sd, selection.Dnai)
		return nil, nil
	}

	DHCPValue, ok := GetFromDHCPMemory(DHCPKey)
	if ok != nil {
		fmt.Println(ok)
	}

	UPFList = upi.sortUPFListByName(UPFList)
	for _, upf := range UPFList {
		logger.CtxLog.Debugf("check start UPF: %s", upi.GetUPFNameByIp(upf.NodeID.ResolveNodeIdToIp().String()))
		pools := getUEIPPool(upf, selection)
		if len(pools) == 0 {
			continue
		}

		sortedPoolList := createPoolListForSelection(pools)
		for _, pool := range sortedPoolList {
			logger.CtxLog.Debugf("check start UEIPPool(%+v)", pool.ueSubNet)
			if DHCPValue.Addr != nil && pool.ueSubNet.Contains(DHCPValue.Addr) {
				logger.CtxLog.Infof("%v", DHCPValue.Addr)
				logger.CtxLog.Infof("Selected UPF: %s", upi.GetUPFNameByIp(upf.NodeID.ResolveNodeIdToIp().String()))
				return upf, DHCPValue.Addr
			}

			addr := pool.allocate()
			if addr != nil{
				key := dhcpValue{
					Addr: addr,
				}
				SetToDHCPMemory(DHCPKey, key)
				logger.CtxLog.Infof("Selected UPF: %s", upi.GetUPFNameByIp(upf.NodeID.ResolveNodeIdToIp().String()))
				return upf, addr
			}

			logger.CtxLog.Debug("check next pool")
		}

		logger.CtxLog.Debug("check next upf")
	}

	logger.CtxLog.Warnf("UE IP pool exhausted for DNN[%s] S-NSSAI[sst: %d sd: %s] DNAI[%s]\n", selection.Dnn,
		selection.SNssai.Sst, selection.SNssai.Sd, selection.Dnai)
	return nil, nil
}

```

In the source code above the function will use 2 function from the dhcpservice functions, which is GetFromDHCPMemory and SetToDHCPMemory. As it is named, the GetFromDHCPMemory is used to get the DHCP information from the MongoDB and SetToDHCPMemory is used to store the DHCP information to the MongoDB. 

```go=
func (ueIPPool *UeIPPool) allocate() net.IP {
	RETRY: 
	allocVal, res := ueIPPool.pool.Allocate()
	if !res {
		logger.CtxLog.Warnf("Pool is empty: %+v", ueIPPool.ueSubNet)
		return nil
	}
	buf := make([]byte, 4)
	binary.BigEndian.PutUint32(buf, uint32(allocVal))
	if DHCPCheck(buf){
		goto RETRY
	}
	logger.CtxLog.Infof("Allocated UE IP address: %v", net.IPv4(buf[0], buf[1], buf[2], buf[3]))
	return buf
}
```

Modifications is also made in the allocate() function to sync up the temporary storage with the mongoDB. By doing this we are able to prevent the posibility of IP duplication in the allocation function. 

## dhcpservice.go
```go=
package context

import (
	"context"
	"log"
	"net"
	"encoding/binary"
	"time"
	"math"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/mongo/options"
	"github.com/free5gc/smf/internal/logger"
)

type dhcpKey struct {
	Selection *UPFSelectionParams `bson:"selection"`
	Supi      string              `bson:"SUPI"`
}

type dhcpValue struct {
	Addr net.IP `bson:"IP_byte"`
}

// MongoDB constants
const (
	mongoURI        = "mongodb://localhost:27017"
	mongoDBName     = "free5gc"
	mongoCollection = "DHCPMemory"
)

var (
	mongoClient *mongo.Client
	dhcpCollection *mongo.Collection
)

func init() {
	// Initialize MongoDB client
	var err error
	mongoClient, err = mongo.NewClient(options.Client().ApplyURI(mongoURI))
	if err != nil {
		log.Fatalf("Failed to connect to MongoDB client: %v", err)
	}

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	err = mongoClient.Connect(ctx)
	if err != nil {
		log.Fatalf("Failed to connect to MongoDB server: %v", err)
	}

	// Get the DHCPMemory collection
	dhcpCollection = mongoClient.Database(mongoDBName).Collection(mongoCollection)
}

func DHCPCheck(ipVal net.IP) bool {
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	err := mongoClient.Ping(ctx, nil)
	if err != nil {
		return false
	}

	filter := bson.M{
		"IP Address": ipVal.String(),
	}

	count, err := dhcpCollection.CountDocuments(ctx, filter)
	if err != nil {
		return false
	}

	return count > 0
}

func GetFromDHCPMemory(cacheKey dhcpKey) (dhcpValue, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    filter := bson.M{
        "SUPI":         cacheKey.Supi,
        "UPFSelection": cacheKey.Selection,
    }

    var result bson.M
    err := dhcpCollection.FindOne(ctx, filter).Decode(&result)
    if err != nil {
        return dhcpValue{}, err
    }

    dhcpAddr := result["IP Address"].(string)
    ipVal := net.ParseIP(dhcpAddr)
    buf := make([]byte, 4)
    octetValue := ipVal.To4()

    var intValue int
    for i := 0; i < 4; i++ {
        intValue += int(octetValue[i]) * int(math.Pow(256, float64(3-i)))
    }
    binary.BigEndian.PutUint32(buf, uint32(intValue))

    dhcpVal := dhcpValue{
        Addr: buf,
    }

    // Renew the expiration date
    expirationDate := time.Now().Add(24 * time.Hour)
    update := bson.M{
        "$set": bson.M{
            "expirationDate": expirationDate,
        },
    }
    _, err = dhcpCollection.UpdateOne(ctx, filter, update)
    if err != nil {
        return dhcpValue{}, err
    }

    return dhcpVal, nil
}

func SetToDHCPMemory(cacheKey dhcpKey, cacheValue dhcpValue) {
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()

	filter := bson.M{
		"SUPI":          cacheKey.Supi,
		"UPFSelection":  cacheKey.Selection,
		"IP Address":    cacheValue.Addr.String(),
	}

	var result bson.M
	err := dhcpCollection.FindOne(ctx, filter).Decode(&result)
	if err != nil && err != mongo.ErrNoDocuments {
		log.Fatalf("Failed to query MongoDB collection: %v", err)
	}

	expirationDate := time.Now().Add(24 * time.Hour)
	// DHCPCheck(cacheValue.Addr.String())
	if result == nil {
		doc := bson.M{
			"SUPI":           cacheKey.Supi,
			"UPFSelection":   cacheKey.Selection,
			"IP Address":     cacheValue.Addr.String(),
			"expirationDate": expirationDate,
		}

		logger.CtxLog.Infof("%v", cacheValue.Addr)

		_, err := dhcpCollection.InsertOne(ctx, doc)
		if err != nil {
			log.Fatalf("Failed to insert document into MongoDB collection: %v", err)
		}
	}
}
```

Finally here is the source code for the DHCP service. This code is a part of a package called "context" and provides functionalities for managing DHCP (Dynamic Host Configuration Protocol) memory using MongoDB. It includes functions for checking the existence of an IP address in the DHCP memory, retrieving a DHCP value based on a key, and setting a DHCP value in the memory based on a key-value pair.

The code initializes a MongoDB client and establishes a connection to the MongoDB server using the provided URI. It also sets up a reference to the DHCPMemory collection within the specified MongoDB database. The package provides a function to check if an IP address exists in the DHCP memory by querying the collection using the IP address as a filter. Additionally, there are functions to retrieve a DHCP value from the memory based on a key and to update the expiration date of the corresponding document in the collection.

The code ensures that the DHCP value retrieved or set in the memory is properly stored in the MongoDB collection. It utilizes BSON encoding/decoding to work with MongoDB documents. The functions handle errors that may occur during database operations and return appropriate values or error messages. Overall, this code enables the management of DHCP memory using MongoDB, allowing efficient storage and retrieval of IP addresses associated with specific keys.

# MongoDB
In this implementation, a new collection will also be generated to store the DHCP information in the MongoDB. To set this up we can follow this step-by-step guide. 

1. Open the MongoDB by using this command
```bash!
mongo
```

2. Then we will use the free5gc database in the MongoDB
```bash
use free5gc
```

3. Create a new collection for storing the DHCP information
```bash
db.createCollection("DHCPMemory")
```

4. Finally we need to make an index to make a feature for expirationTime for the leased IP. We will use the TTL index feature of the mongoDB.We will make a key that will store the expiration time for the leased IP. For that we can use this command. We can modify the duration of the lease time in the dhcpservice.go source code. 
```bash
db.DHCPMemory.createIndex({ "expirationDate": 1 }, { expireAfterSeconds: 0 })
```

5. We can use the db.DHCPMemory.find() to see the stored DHCP information stored in the MongoDB
:::     info
![](https://hackmd.io/_uploads/HJUtKj7Fn.png)
:::