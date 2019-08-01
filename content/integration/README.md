

## Requirements
There are few requirements for the exchanges to be able to be a part of Warp Network. <br/>
The processing itself done transparently for the exchange through GEO Node, 
but there are several integrations that should be done, for proper communication between GEO Node, 
and exchange back-end. 

#### Support assets addresses discovering
In case if some user wants to withdraw his balance 
and transfer it to another exchange through Warp Network — 
two actions should be performed by the exchange, that initialises the operation:

1. Lookup for entered BTC/ETH/etc address in the Warp Network.
1. In case if there is an exchange that approved address presence — send assets.  

