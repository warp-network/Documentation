

# Integration flow
There are few requirements for the exchanges to be able to be a part of Warp Network. <br/>
The processing itself done transparently for the exchange through GEO Node, 
but there are several integrations that should be done, for proper communication between GEO Node, 
and exchange back-end. 

<br/>
<br/>

## Assets addresses discovering flow
In case if some user wants to withdraw his balance 
and transfer it to another exchange through Warp Network — 
two actions should be performed by the exchange, that initialises the operation:

1. Lookup for entered BTC/ETH/etc address in the Warp Network.
1. In case if there is an exchange that approved address presence — send assets.  

Here is the flow for address resolving:
<br/>
<br/>

_Figure 1: Initial state. Exchange A initiates sending operation to exchange B._ <br/>
![image](https://github.com/warp-network/Documentation/blob/master/content/integration/img/1.png?raw=true)

<br/>
<br/>

_Figure 2: Exchange A asks router of the Hub for address resolving._ <br/>
![image](https://github.com/warp-network/Documentation/blob/master/content/integration/img/2.png?raw=true)

<br/>
<br/>

_Figure 3: Router of the Hub resolves the address received._ <br/>
![image](https://github.com/warp-network/Documentation/blob/master/content/integration/img/3.png?raw=true)

<br/>
<br/>

_Figure 4: Router of the Hub reports final GEO Node address that is capable to accept the payment._ <br/>
![image](https://github.com/warp-network/Documentation/blob/master/content/integration/img/4.png?raw=true)

<br/>
<br/>

## Router API Description

**Request** 

`curl -X GET "https://router.warp.geoprotocol.io/api/v1/addresses/ability/?address=<some address>&ticker=btc"`

* `ticker` could be one of `btc`, `eth`.
* `address` BTC/ETH address. 

```json
{
   "acceptable": true,
   "market_place": {
       "name": "<exchange title>",
       "interface": "<GEO Node IP Address and Port>"
   }
}
```

**Response scheme/example**

```json
{
    "acceptable": true,
    "market_place": {
        "name": "kuna",
        "interface": "157.230.100.213:2000"
   }
}
```

```json
{
    "acceptable": false,
    "market_place": {
        "name":"",
        "interface":""
    }
}
```

<br/>
<br/>

## Exchanges API Requirements
To be able to react on router's requests, exchange should implement some simple API.
The main reason behind this API is to inform router (and other exchanges through it) 
about presence or absence of the requested address in ecosystem of current exchange.

`/api/warp/v1/addresses/ability/?address=<some address>&ticker=<ticker>`

Parameters are equal to router's API.

**Responses expected**

OK, requested address belongs to the current exchange. 
```json
{
    "acceptable": true
}
``` 

<br/>

Requested address doesn't belong to the current exchange.
```json
{
    "acceptable": false
}
``` 

<br/>
<br/>

## Assets sending/receiving flow
This set of API methods is expected to be used by the back-end of the exchange. <br/>
In case if router returned positive response — assets sending transaction should be issued 
via Warp Network.

### Assets sending
Assets sending should be done in 2 stages:
1. Max flow checking — due to the nature of state channels networks, 
it is required to verify present payment flow between sender and receiver.
1. Payment issuing — in case if max flow reported >= expected operation amount — 
operation should be started. Otherwise — insufficient funds error should be reported for the user. 
   
**API methods of the node, that should be used in this flow:**
* [Max Flow Prediction](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#max-flow-predicition)
* [Payment](https://github.com/GEO-Protocol/Documentation/blob/master/client/api-http/README.md#payments--transactions-issuing)

**WARN: Payments must contains payload with asset address!**
Transaction payload should be set via corresponding parameter `payload` in JSON format:
```json
{
    "ticker": "eth",
    "address":"0xf42a1aa4a83020470D96AC47801fF80EEdC36145"
}
```

**[Optional] Node API Key and IP Address pinning**
It is expected, that GEO Node is located in trusted environment that is accessible by the back-end of the exchange. 
For the cases when additional security level is needed — there is an ability to specify API-key
for the [node CLI](https://github.com/GEO-Protocol/Documentation/tree/master/client/), and to pin host IP address 
(requests would be processed only in case if corresponding HTTP header would contain expected IP address).
 To configure this parameters — please, refer to the [node CLI](https://github.com/GEO-Protocol/Documentation/tree/master/client/) configuration file.

### Assets receiving
Back-end of the exchange should check for incoming payments from time to time. <br/>
Recommended time-out is `3-4 seconds`.

In case if incoming payments occurred — it is expected that balance of corresponding address 
would be updated by the amount of the operation. Corresponding address is available in payload of the transaction.

**Example:**
```json
{
    "ticker": "eth",
    "address": "0xf42a1aa4a83020470D96AC47801fF80EEdC36145"
}
``` 

**API methods of the node that should be used in this flow:**
* [Payments History](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#payments-history)