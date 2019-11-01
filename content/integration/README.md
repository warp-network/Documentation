

# Integration flow
There are few requirements for the exchanges to be able to be a part of [Warp Network](https://github.com/warp-network). <br/>
The processing itself done transparently for the exchange through [GEO Node](https://github.com/GEO-Protocol/GEO-network-client), 
but there are several integrations that should be done, for proper communication between [GEO Node](https://github.com/GEO-Protocol/GEO-network-client), 
and exchange back-end. 

<br/>
<br/>

## Assets addresses discovering flow
In case if some user wants to withdraw his balance 
and transfer it to another exchange through [Warp Network](https://github.com/warp-network) — 
two actions should be performed by the exchange, that initialises the operation:

1. Lookup for entered `btc` / `eth`  address in the [Warp Network](https://github.com/warp-network).
1. In case if there is an exchange, that has been approved address ownership — send assets to it.  

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

_Figure 4: Router of the Hub reports final [GEO Node](https://github.com/GEO-Protocol/GEO-network-client) address that is capable to accept the payment._ <br/>
![image](https://github.com/warp-network/Documentation/blob/master/content/integration/img/4.png?raw=true)

<br/>
<br/>

## Router API Description

**Request** 

`curl -X GET "https://router.warp.geoprotocol.io/api/v1/addresses/ability/?address=<some address>&ticker=btc"`

* `ticker` could be one of `btc`, `eth`.
* `address` btc/eth address. 

```json
{
   "acceptable": true,
   "market_place": {
       "name": "<human readable exchange title>",
       "interface": "<GEO Node IPv4 address AND port>"
   }
}
```

**Response scheme/example**

Some exchange has approved address ownership.
```json
{
    "acceptable": true,
    "market_place": {
        "name": "Super Exchange",
        "interface": "157.230.100.213:2000"
   }
}
```

No one exchange has approved the address entered.
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
via [Warp Network](https://github.com/warp-network).

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
It is expected, that [GEO Node](https://github.com/GEO-Protocol/GEO-network-client) is located in trusted environment that is accessible by the back-end of the exchange. 
For the cases when additional security level is needed — there is an ability to specify API-key
for the [node CLI](https://github.com/GEO-Protocol/Documentation/tree/master/client/), and to pin host IP address 
(requests would be processed only in case if corresponding HTTP header would contain expected IP address).
 To configure this parameters — please, refer to the [node CLI](https://github.com/GEO-Protocol/Documentation/tree/master/client/) configuration file.

### Assets receiving
When registering an exchange, it specifies a callback URL at which we will notify it of the incoming payment. This URL will have the following parameters:

* `transaction_uuid` - transaction uuid to identify all payments;
* `address` - wallet address of the user who is receiving the payment;
* `ticker` - payment currency;
* `source` - name of the exchange from which the payment was received;
* `amount` - payment amount.

The `contractor_address` parameter of the payment must contain the address of the exchange that it gets from the router, as well as the address type (in this case, the address type will always be 12). For example, the router gave the address of the exchange 152.46.12.200, then `contractor_address` should take the value of 12-152.46.12.200

The `amount` parameter is the amount that will be sent to another exchange, but because it is an integer type, it must be specified in minimum indivisible units of equivalent. For example, the minimum indivisible unit in BTC is Satoshi = 0.00000001 BTC, so amount = 1 is 1 Satoshi, respectively amount = 100000000 is 1 BTC

### Balance checking
[This method](https://github.com/GEO-Protocol/Documentation/blob/master/client/api-http/README.md#get-trust-line-by-contractor-id) with the parameter `contractor_id` = 0 is used to check the balance of a node. The current balance of the exchange is determined by the balance parameter in the response.

The maximum amount in a given equivalent that an exchange can transfer to another exchange is determined by its current balance in that equivalent.

**API method of the node that should be used in for retrieving history of transactions:**
* [Payments History](https://github.com/GEO-Protocol/Documentation/tree/master/client/api-http#payments-history)

**GEO Protocol uses the following equivalents:**
* BTC - 1001
* ETH - 1002