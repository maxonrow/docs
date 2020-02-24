# MsgTypeEndorsement

This is the message type used to endorse an item of a non-fungible token.

#### Parameters
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Endorser| | 
| itemID | string | true   | Item ID, which must be unique| | 


#### Example
```
{
    "type": "nonFungible/endorsement",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1k9sxz0h3yeh0uzmxet2rmsj7xe5zg54eq7vhla",
        "itemID": "ITEM-123"
    }
}


```

#### Listening to events of MsgTypeEndorsement
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](/en/latest/pic_module/MsgTypeEndorsement.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using EndorsedNonFungibleItem(string,string,string)
* Endorser
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Endorser| | 
| itemID | string | Item ID| | 


