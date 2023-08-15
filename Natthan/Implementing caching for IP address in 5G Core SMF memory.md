# Introduction
By integrating DHCP with SMF and utilizing the SMFContext package, administrators can enhance the management, reliability, and persistence of the DHCP service. The SMFContext package allows for the storage of DHCP configuration and state information within the SMF framework, enabling easy access, manipulation, and monitoring through standard SMF tools. This integration streamlines the management process, promotes consistency, and ensures critical DHCP data is persistently maintained, resulting in a more efficient and reliable network environment.

# Implementation
To implement DHCP in SMF and utilize the SMFContext package, the following changes need to be made in key files: context.go, pdu_session.go, and user_plane_information.go. Additionally, a new code is created in dhcpservice.go.

## context.go
![](https://hackmd.io/_uploads/SJVPtN5Fh.png)

A new variable is added in the SMFContext to store the DHCP information. The data type will be set sync.Map.The benefit of using sync.Map in Go is its built-in thread-safe concurrent map operations, providing efficient and safe access to shared data without the need for external synchronization mechanisms.

## pdu_session.go
![](https://hackmd.io/_uploads/SyAGnVqYn.png)

A modification is made in calling the SelectUPFAndAllocUEIP by adding a new parameter to be passed, which is the UE identifier SUPI. 

## user_plane_information.go
```go!
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

	DHCPValue, ok := GetFromDHCPMemoryLocal(DHCPKey)
	if !ok {
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
					ExpiresAt: time.Now().Add(24 * time.Hour),
				}
				SetToDHCPMemoryLocal(DHCPKey, key)
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
The function is modified so it would call the DHCP functions to save the related information in the SMF context

## dhcpservice.go
```go!
import (
	"fmt"
	"net"
	"time"
)

type dhcpKey struct {
	Selection *UPFSelectionParams 
	Supi      string             
}

type dhcpValue struct {
	Addr net.IP `bson:"IP_byte"`
	ExpiresAt time.Time 
}


func SetToDHCPMemoryLocal(cacheKey dhcpKey, cacheValue dhcpValue) {
	DHCPString := cacheKey.Supi + cacheKey.Selection.Dnn + cacheKey.Selection.Dnai + fmt.Sprint(cacheKey.Selection.SNssai.Sst) + cacheKey.Selection.SNssai.Sd
	smfContext.DHCPMemory.Store(DHCPString, cacheValue)
}



func GetFromDHCPMemoryLocal(cacheKey dhcpKey) (dhcpValue, bool) {
	key := cacheKey.Supi + cacheKey.Selection.Dnn + cacheKey.Selection.Dnai + fmt.Sprint(cacheKey.Selection.SNssai.Sst) + cacheKey.Selection.SNssai.Sd

	DHCPValue, ok := smfContext.DHCPMemory.Load(key)
	if !ok {
		return dhcpValue{}, false
	}

	dhcpVal := DHCPValue.(dhcpValue)

	// Check if the DHCP information has expired
	if time.Now().After(dhcpVal.ExpiresAt) {
		// Delete the expired DHCP information
		smfContext.DHCPMemory.Delete(key)
		return dhcpValue{}, false
	}

	// Update the expiration time
	dhcpVal.ExpiresAt = time.Now().Add(24 * time.Hour)
	smfContext.DHCPMemory.Store(key, dhcpVal)

	return dhcpVal, true
}

```
This code implements a DHCP memory caching mechanism. It defines two structs: `dhcpKey` represents the key used to store and retrieve values, while `dhcpValue` holds the actual data, including an IP address and expiration time. The `SetToDHCPMemoryLocal` function stores a `dhcpValue` in the memory cache using a constructed key string. On the other hand, the `GetFromDHCPMemoryLocal` function retrieves a `dhcpValue` from the cache based on a given key. It performs checks to ensure the data is valid and not expired, updating the expiration time if necessary. If the data has expired, it is removed from the cache.

In summary, this code provides a way to store and retrieve DHCP-related information using a memory cache. It ensures the data is up-to-date and removes expired entries. This mechanism can be useful for efficient and quick access to DHCP data in networking applications.

# Results

:::success
![](https://hackmd.io/_uploads/SJF56N5Kh.png)
:::

The result above shows that the IP will be allocated based on the SUPI of the UE and the UPF Selection Parameters Passed

![](https://hackmd.io/_uploads/rJSLATb2n.png)


