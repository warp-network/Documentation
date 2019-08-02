

# Integration flow
There are few requirements for the exchanges to be able to be a part of Warp Network. <br/>
The processing itself done transparently for the exchange through GEO Node, 
but there are several integrations that should be done, for proper communication between GEO Node, 
and exchange back-end. 

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

<hr>

## Router API Description
#### Request
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

Response Example:
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

<hr>

## Exchanges API Requirements
To be able to react on router's requests, exchange should implement some simple API.
The main reason behind this API is to inform router (and other exchanges through it) 
about presence or absence of the requested address in ecosystem of current exchange.

`/api/warp/v1/addresses/ability/?address=<some address>&ticker=<ticker>`

Parameters are equal to router's API.

**Responses expected:**

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