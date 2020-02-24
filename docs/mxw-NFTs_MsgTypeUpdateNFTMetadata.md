# MsgTypeUpdateNFTMetadata

This is the message type used to update metadata of a non-fungible token.


#### Parameters
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Token owner| | 
| metadata | string | true   | Metadata of token| | 

#### Example
```
{
    "type": "nonFungible/updateNFTMetadata",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "metadata": "token metadata updated here"
    }
}
```


#### Listening to events of MsgTypeUpdateNFTMetadata
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](/en/latest/pic_module/MsgTypeUpdateNFTMetadata.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UpdatedNonFungibleTokenMetadata(string,string)
* Token owner
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 



