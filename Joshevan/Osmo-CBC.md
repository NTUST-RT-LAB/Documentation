# OsmoCBC
**Osmocom is a Free and Open Source Software (FOSS) community that develops and maintains a variety of software (and partially also hardware) projects related to mobile communication. OsmoCBC is the Osmocom implementation of a Cellular Broadcast Centre (CBC). It is the heart of the Cell Broadcast Service (CBS) as well as a variety of disaster/emergency warning systems (PWS).**

OsmoCBC provides a REST/JSON interface for receiving cell broadcast messages from external entities and a 3GPP CBSP interface towards BSCs and also the MME interface using the SBc-AP protocol, it also provides telnet-based command line interface for configuration and introspection called VTY.
In this documentation, we will try to install OsmoCBC and implements it

## OsmoCBC Installation
* Install library dependency first (libosmocore)
    * Clone the library from remote repository (gitea)
    ```
    git clone https://gitea.osmocom.org/osmocom/libosmocore.git
    ```
    
    ![](https://i.imgur.com/a9FUpBU.png)

    * Compile and Install the library
    ```
    cd libosmocore/
    autoreconf -i
    ./configure
    make
    sudo make install
    sudo ldconfig -i
    cd ..
    ```
    ![](https://i.imgur.com/qCXxIOR.png)
    
    ![](https://i.imgur.com/xckowRp.png)

    ![](https://i.imgur.com/qCYrvkx.png)
    
    ![](https://i.imgur.com/q3jDLcy.png)

    ![](https://i.imgur.com/IHwCIa9.png)
    
* Clone the source code from remote repository (gitea)
```
git clone https://gitea.osmocom.org/cellular-infrastructure/osmo-cbc
```
![](https://i.imgur.com/eolZO8K.png)

* Compile and Install from the source code
```
cd osmo-cbc/
autoreconf --install
./configure
make
sudo make install
sudo ldconfig -i
cd ..
```
![](https://i.imgur.com/azQ64WI.png)

![](https://i.imgur.com/OT5Iiof.png)

![](https://i.imgur.com/MynkzhQ.png)

![](https://i.imgur.com/TzYpRhY.png)

![](https://i.imgur.com/VtPjeQd.png)

![](https://i.imgur.com/8VDymI0.png)

:::info
Detail for the command:
* `autoreconf --install`: Automatically create a configure file, The --install option will add some additional files needed for the configure script.
* `./configure`: Checks system for the required software needed to build the program and create makefiles and configurations for the software that you are about to compile and install
* `make`: Compile the software package
* `sudo make install`: Install the package in our system, default directory of the installed software is /usr/local/bin
* `sudo ldconfig -i`: creates  the necessary links and cache to the most recent shared libraries found in the directories specified on the command line, in the file /etc/ld.so.conf, and in the trusted  directories
:::

::: warning
If there is error messages when building program, it indicates that we need to install certain packages for our system. It can be done using the command
```
sudo apt-get install *package name*
```
If the required packages is not available in Linux Repository, we should manually install it like the libosmocore library
:::

* Testing if the OsmoCBC works
```
osmo-cbc --version
```
![](https://i.imgur.com/u8Y0200.png)

```
cd doc/examples/osmo-cbc
osmo-cbc
```
![](https://i.imgur.com/afNwOUw.png)
:::info
The osmo-cbc command should be run with the config file (osmo-cbc.cfg), it can be done by adding -c options. If the config file is not specified, it will usethe config file in the working directory. This working directory (doc/examples/osmo-cbc) has a config file
:::

* Trying to use the VTY interface by using telnet 
```
telnet 127.0.0.1 4264
```
![](https://i.imgur.com/lxnBFyA.png)

## OsmoCBC Implementation
In this part, we will try to implement OsmoCBC by trying to send a CBS message and see wheteher the message can be seen in the OsmoCBC.
:::info
For sending the CBS message, we will use `cbc-apitool.py`, a simple python program to test the OsmoCBC Rest API for creating/deleting CBS message
:::

Steps to send a CBS message to OsmoCBC:
* Run the Osmo-CBC
```
cd doc/examples/osmo-cbc
osmo-cbc
```
![](https://hackmd.io/_uploads/Hkff-LXE2.png)
:::info 
Same as before, we run the osmo-cbc inside the directory where there is a configuration file (osmo-cbc.cfg). This directory (doc/examples/osmo-cbc) contains an example configuration file. If desired, we can create our own configuration file for OsmoCBC to use
:::
* Run the VTY interface via telnet by opening a new terminal session
```
telnet 127.0.0.1 4264
```
![](https://hackmd.io/_uploads/H1HkGUQNn.png)
:::info
VTY (Virtual Tele Type) is a command line interface to perform interaction easily. The VTY has the concept of nodes and commands. Each command can consist out of several words followed by a variable number of parameters. 
:::

* Open a new terminal session to run the `cbc-apitool.py`
```
cd contrib/
python3 cbc-apitool.py
```
![](https://hackmd.io/_uploads/SkvrQ8QN3.png)

:::info
The `cbc-apitool.py` is located inside the contrib folder. As we can see, we should specify arguments to create a CBS/ETWS message
:::

### Creating a CBS message
* Trying to create a CBS message 
```
python3 cbc-apitool.py -H 127.0.0.1 -p 12345 create-cbs --msg-id 2000 --payload-data-utf8 "test message"
```
![](https://hackmd.io/_uploads/r1SSE8mNn.png)

:::info
Details for argument:
`-H` is for host, we use 127.0.0.1 as described in the configuration file
`-p` is for port, we use 12345 as descirbed also in the configuration file
`create-cbs` is to create a CBS message
`--msg-id` is for CBS message id range from 0 to 65535, we try to use 2000
`--payload-data-utf8` is for the CBS message payload, we try to input "test message"
:::
:::warning
If we want to change the host and port address, we can change the configuration address of OsmoCBC. The host and port address can be seen when we start OsmoCBC:
![](https://hackmd.io/_uploads/Hki3VIXN3.png)
:::
:::warning
Details for the CBS message:
```
python3 cbc-apitool.py create-cbs -h
```
![](https://hackmd.io/_uploads/SyllOvU7V3.png)

The needed arguments is MSG_ID and PAYLOAD_DATA_UTF8, the rest is optional.
Actually the CBS message schema has more attributes, but most of it is hardcoded in the `cbc-apitool.py`. If we want to change the value of the attributes, we can change the code or if we want to see the complete schema, we can see the file `cbc.schema.json`
:::

* Show the CBS message in OsmoCBC through the VTY interface
```
show messages cbs
```
![](https://hackmd.io/_uploads/Skdbu8XEh.png)

:::success
As we can see, the message is successfuly received. The MsgID is 07D0, which is the hexadecimal number of 2000
::: 

* Show the detail of the message
``` 
show message  id 2000
```
![](https://hackmd.io/_uploads/SJxC5UmV2.png)
:::success
We can see that the message is successfully received, but we have not configured the BCS/MME, so that the number of brodcasts completed is empty
:::

### Creating an ETWS message
*  Trying to create an ETWS message
```
python3 cbc-apitool.py -H 127.0.0.1 -p 12345 create-etws --msg-id 1000
```
![](https://hackmd.io/_uploads/HymA3ImN3.png)
:::info
The command is almost the same with CBS, but ETWS only needs message id
:::

*  Show the ETWS message in OsmoCBC through VTY interface
```
show messages etws
```
![](https://hackmd.io/_uploads/Hyz4aL7Vh.png)
:::success
As we can see, the message is successfuly received. The MsgID is 03E8, which is the hexadecimal number of 1000
::: 

* Show the detail of the message
``` 
show message id 1000
```
![](https://hackmd.io/_uploads/Bk4u6U7Nn.png)
:::success
We can see that the message is successfully received, but we have not configured the BCS/MME, so that the number of brodcasts completed is empty
:::
:::spoiler Difference between CBS and ETWS message
The difference between CBS and ETWS message can be seen from the message schema, we try to see the schema from the `cbc-apitool.py`
```
cd osmo-cbc/contrib/
nano cbc-apitool.py
```
![](https://hackmd.io/_uploads/BJ_2Kmt43.png)

![](https://hackmd.io/_uploads/HJx19XYEn.png)

Search for the CBS and ETWS message part to see the difference:
![](https://hackmd.io/_uploads/ryvVomKV3.png)

![](https://hackmd.io/_uploads/B117s7FV2.png)
:::info
From the message, we can see that the difference between CBS and ETWS message is located in the payload. The payload for CBS is only payload data, but payload for ETWS is spesifically for warning, like the "warning_type", "emergency_user_alert", and "popup_on_display".
For the complete details of message, either CBS or ETWS can be seen on the smscb_schema.json file located in the osmo-cbc directory
:::

:::spoiler Steps to send CBS/ETWS message without using `cbc-apitool.py`
We use `cbc-apitool.py` to easily send CBS/ETWS message. If we do not want to use it, we must access the API endpoint manually
* POST /api/ecbe/v1/message
This command is used to create a new SMSCB or ETWS message inside the CBC. The cbc_messsage type as specified in
the JSON schema

* DELETE /api/ecbe/v1/message/:message_id
This command is used to delete an existing SMSCB or ETWS message from the CBC. The :message_id parameter is the decimal integer representation of the cbc_message.smscb.message_id that was specified when creating the message via the POST command stated above.

We can use REST client software, like curl or postman, to access those APIs. We also must to change the ECBE IP Address configuration to make it accessible outside the host.
:::

## Try to implement MME
MME (Mobile Management Entity) is the control plane for the UE to access 4g network. In CBC network architecture, the CBS message will also be forwared to MME, so we will try to implement the MME
To implement the MME, we also use software from osmocom called osmo-uecups. Osmo-uecups is a simulator for the MME side of GTP-U (GPRS Tunneling Protocol). The osmo-uecups is not primarily intended for MME or MME-CBC, so we will try it first.

### Osmo-uecups Installation
* Clone the source code from remote repository (gitea)
```
git clone https://gitea.osmocom.org/cellular-infrastructure/osmo-uecups
```
![](https://hackmd.io/_uploads/r1FA0MtV2.png)

* Compile and Install from the source code
```
cd osmo-cbc/
autoreconf --install
./configure
make
sudo make install
sudo ldconfig -i
cd ..
```
![](https://hackmd.io/_uploads/H1SeJXtNh.png)

![](https://hackmd.io/_uploads/rJ_e1XtEh.png)

![](https://hackmd.io/_uploads/r1oxkXYN2.png)

![](https://hackmd.io/_uploads/SyTlJ7tN3.png)

![](https://hackmd.io/_uploads/H1xW1QKEh.png)


:::info
The commands used for instalattion is the same with the command to install osmocom software above.
:::

* Running osmo-uecups
```
cd osmo-uecups/doc/examples
osmo-uecups-daemon
```
![](https://hackmd.io/_uploads/ByY61XKV2.png)

:::success
The osmo-uecups is successfully running in the port 4268
:::

### Trying to do SBc-AP protocol between Osmo-CBC and Osmo-uecups

* Configure the Osmo-CBC config file
```
cd osmo-cbc/doc/examples/osmo-cbc/
nano osmo-cbc.cfg
```
![](https://hackmd.io/_uploads/ryWF-QFEn.png)

![](https://hackmd.io/_uploads/BkTzS7KEn.png)
* Configure the Config file to connect with osmo-uecups
    * Delete the IPv6 from sbcap and peer sbcap
    * Change the remote-ip in sbcap to 127.0.0.1 and remote-port to 4268
    ![](https://hackmd.io/_uploads/Hkw3r7tEh.png)
    * Exit and save the configuration
:::info
We change the peer sbcap to the IP Address and port number of osmo-uecups.
The osmo-uecups is running on the 127.0.0.1, so we change the remote-ip to it and from the documentation, we know that osmo-uecups runs SCTP protocol at port 4268, so we also change the remote-port to it
:::

* Run the Osmo-uecups daemon
```
cd osmo-uecups/doc/examples
osmo-uecups-daemon
```
![](https://hackmd.io/_uploads/HJOs8XKE3.png)

* Open a new terminal session and run the osmo-cbc
```
cd osmo-cbc/doc/examples/osmo-cbc/
osmo-cbc
```
![](https://hackmd.io/_uploads/HyQnPQtVh.png)
:::success
We can see that the example-mme is connected and in the osmo-uecups daemon:
![](https://hackmd.io/_uploads/rkC_DQK42.png)
The osmo-uecups daemon also successfully accept a new connection
:::

* Try to send a CBS message using `cbc-apitool.py` to see whether the CBS message can be forwared into the MME
```
cd osmo-cbc/contrib/
python3 cbc-apitool.py -H 127.0.0.1 -p 12345 create-cbs --msg-id 2000 --payload-data-utf8 "test message"
```
![](https://hackmd.io/_uploads/SJhmumtEn.png)

In the osmo-uecups daemon:
![](https://hackmd.io/_uploads/Hy1ddQYE3.png)
:::danger
The message is failed to be forwarded into the mme as it said that error decoding JSON.
:::

* Try to send a CBS message manually using cURL, to post the data manually using JSON format
```
curl -X POST -H "Content-Type: application/json" -d '{
  "cbe_name": "cbc_apitool",
  "category": "normal",
  "repetition_period": 1,
  "num_of_bcast": 1,
  "scope": {
    "scope_plmn": {}
  },
  "smscb_message": {
    "message_id": 100,
    "serial_nr": {
      "serial_nr_decoded": {
        "geo_scope": "plmn_wide",
        "msg_code": 1,
        "update_nr": 1
      }
    },
    "payload": {
      "payload_decoded": {
        "character_set": "gsm",
        "data_utf8": "test"
      }
    }
  }
}' 127.0.0.1:12345/api/ecbe/v1/message

```
![](https://hackmd.io/_uploads/HJD5mAkHn.png)
:::info
cURL is used to send a HTTP request to an API endpoint. Details for the commands:
- `-X POST`: to set the HTTP method to POST
- `-H "Content-Type: application/json"`to set request header to JSON
- `-d ''`: specify the data
- `127.0.0.1:12345/api/ecbe/v1/message` the API endpoint as specified above
:::
The output in osmo-uecups:
![](https://hackmd.io/_uploads/B1QoSAkr2.png)
:::danger
The messages still failed to be forwared into the osmo-uecups
:::
* Try to send a message using SCTP client directly to osmo-uecups, without osmoCBC
    * Install SCTP library for python from remote repository
    ```
    git clone https://github.com/P1sec/pysctp/blob/master/_sctp.c
    ```
    ![](https://hackmd.io/_uploads/Hyt6IA1Hn.png)
    
    ```
    cd pysctp
    sudo apt-get install python3-setuptools
    sudo apt-get install python3-dev
    sudo python3 setup.py install
    ```
    ![](https://hackmd.io/_uploads/SkX7w01Bh.png)
    ![](https://hackmd.io/_uploads/ByZNPCJB2.png)
    ![](https://hackmd.io/_uploads/rJowPAyBn.png)
    ![](https://hackmd.io/_uploads/S1NswR1H3.png)
    
    ![](https://hackmd.io/_uploads/rJS2P0ySh.png)    
    :::info
    These commands will install required dependecy to add library from source code to python, then install the library itself for python to use
    :::
    * Download SCTP client code from https://nickvsnetworking.com/sctp-in-python/
    
    ![](https://hackmd.io/_uploads/r1b7u0kH2.png)
    ![](https://hackmd.io/_uploads/SkzFd0JB3.png)
    :::info
    We can see that the sctp_client code is downloaded
    :::
    * Configure the`sctp_client.py`
    ```
    nano sctp_client.py
    ```
    ![](https://hackmd.io/_uploads/B16YYAJH3.png)

    ![](https://hackmd.io/_uploads/HyMYKC1Bn.png)
    
    Send the message:
    ```
    python3 sctp_client.py
    ```
    ![](https://hackmd.io/_uploads/H1J_2AJSh.png)
    
    Output at osmo-uecups:
    ![](https://hackmd.io/_uploads/B15B9CkSh.png)
    :::warning
    We can see that the error message is same, but this time, the message is shown in the Rx. The error is caused by the message in `sctp_client.py` is not in JSON format
    :::

    * Configure the `sctp_client.py` again to change the message into CBS message format
    ```
    nano sctp_client.py
    ```
    ![](https://hackmd.io/_uploads/B16YYAJH3.png)
    
    ![](https://hackmd.io/_uploads/HyWf3RyBh.png)

    * Sending again the message
    ```
    python3 sctp_client.py
    ```
    ![](https://hackmd.io/_uploads/H1J_2AJSh.png)
    
    Output in osmo-uecups:
    ![](https://hackmd.io/_uploads/r1Osn0kS2.png)
    :::info
    The message is received, but the error message is "Unknown command received"
    :::
* Examining the code at osmo-uecups to see why this happens
    * Examine cups_client.c
    
    ![](https://hackmd.io/_uploads/r1p5001rn.png)
    :::info
    From that line of codes, can be known that the message will be printed first in the command line (line 531), after that it will parse the JSON and it will handle the JSON (line 542). From this, we know that the error when connecting OsmoCBC and Osmo-uecups is not caused by error JSON format, but the message cannot be forwarded, so the Rx shows an empty message and error decoding JSON because of empty message.
    :::
    Examine cups_client_handle_json function:
    ![](https://hackmd.io/_uploads/rkU8y1lBn.png)
    :::info
    From that line of codes, can be known that the JSON message that is received will be checked, and if the JSON does not contain the correct message for osmo-uecups, it will return "Unknown command received", like the one that we received. 
    :::
:::info
Conclusion: Osmo-uecups cannot be used to implement MME fully because it is not primarily intended for MME or MME-CBC. Osmo-uecups is primarily intended to implement the GTP-U, so it can only received command (in JSON format) to implement the GTP-U. Nevertheless, it can be used to show Sbc-AP connectivity between CBC and MME, although it cannot decode the message. The reason why the message sent from Osmo-CBC to Osmo-uecups is empty is maybe because the handler when receiving CBS message from REST API still not correct.
:::


## CBCF broadcast

### The difference between CBC and CBCF broadcast procedure
The broadcast procedure between CBC and CBCF is almost the same, the difference is :
- CBCF use Namf (N50) interface between CBCF and the AMF and use HTTP/2 Protocol, but CBC use Sbc interface between CBC and the MME and use SBC-AP Protocol
- CBCF use N2 interface between AMF and NG-RAN, but CBC use S1-MME interface between MME and E-UTRAN

The detail of the warning message delivery procedure can be seen [here](https://hackmd.io/@joshevan/SJoZvlpQh#9-Warning-Message-in-E-UTRAN-and-NG-RAN)

:::info
HTTP/2 (Hyper Text Transfer Proocol 2) method is an internet protocol for communicating between client and server using HTTP method, like GET, POST, PUT, DELETE, etc. HTTP/2 has a few improvements in performance, efficiency, speed from its previous version (HTTP/1.1). One of the reason is because HTTP/2 implement multiplexing, so it allows multiple request and response to be sent on a single TCP connection and it reduces latency and increase performance 
:::

### Write-replace-warning-request in OsmoCBC
To know how the write-replace-warning-request works in OsmoCBC, we must see at the Sbc-AP connectivity because it is the connectivity between the OsmoCBC and the MME, to see it we must see the details of the code that implements Sbc-AP connectivity.
* Search for the code that implements the Sbc-AP connectivity:
```
cd osmo-cbc/src
ls
```
![](https://hackmd.io/_uploads/S1GZ-VKEh.png)

*  We can see that there are some codes, we try to examine the sbcap_link.c 

```
nano sbcap_link.c
```
![](https://hackmd.io/_uploads/rJecZ4KVn.png)
![](https://hackmd.io/_uploads/HyUjZVtE2.png)

:::info
After examining this file, there are few functions from this file:


* cbc_sbcap_link_cli_connect_cb: Function when CBC as a SCTP client successfully connect to a Sbc-AP link (MME)
![](https://hackmd.io/_uploads/SkoMz_frn.png)

*  cbc_sbcap_link_cli_read_cb: Function when data is received CBC as a SCTP client from a Sbc-AP link (MME), it will decode and process the message
![](https://hackmd.io/_uploads/rJJvfOMSn.png)
   
*  cbc_sbcap_link_open_cli: Function to open a SCTP client connection to an SBc-AP link.
![](https://hackmd.io/_uploads/BkgfQOMHh.png)

*  sbcap_cbc_srv_read_cb: Function when data is received CBC as a SCTP server from a Sbc-AP link (MME), it will decode and process the message
![](https://hackmd.io/_uploads/rkxL7uGB3.png)


*  sbcap_cbc_accept_cb: Function when a new SCTP connection to an SBc-AP link is accepted by the server.
![](https://hackmd.io/_uploads/r1Pu7OGBh.png)

* cbc_sbcap_link_tx: Function to send the CBS message to the SBc-AP link
![](https://hackmd.io/_uploads/B1v0SuzBn.png)

:::

* Examine the sbcap_msg.c
:::info
Overall, this code is responsible for generating SBc-AP messages for writing, replacing, or stopping warning broadcasts based on the provided input. After examining this file, there are few functions from this file:


* msgb_put_sbcap_cell_list: Function to put Cell List parameters into the CBC Message by inputting the PLMN id
![](https://hackmd.io/_uploads/B1OLcvjI3.png)

* sbcap_gen_write_replace_warning_req: Function to request a write replace warning request, this function will input some parameters for the write replace warning request message and this function will be called if the CBC want to genereate a write replace warning request message to sent to the MME
![](https://hackmd.io/_uploads/r1rhhPjI3.png)

* sbcap_gen_stop_warning_req: Function to request a stop warning request, this function will input some parameters for the write stop warning request message and this function will be called if the CBC want to genereate a stop replace warning request message to sent to the MME
![](https://hackmd.io/_uploads/HyhZ6DiUn.png)