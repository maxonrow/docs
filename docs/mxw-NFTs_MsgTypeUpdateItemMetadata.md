# MsgTypeUpdateItemMetadata

This is the message type used to update item metadata of a non-fungible token.



#### Parameters
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Item owner| | 
| itemID | string | true   | Item ID| | 
| metadata | string | true   | Metadata of item| | 


#### Example
```
{
    "type": "nonFungible/updateItemMetadata",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
        "itemID": "ITEM-123",
        "metadata": "update Item metadata 9991"
    }
}
```

#### Listening to events of MsgTypeUpdateItemMetadata
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](/en/latest/pic_module/MsgTypeUpdateItemMetadata.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UpdatedNonFungibleItemMetadata(string,string,string)
* Item owner
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Item owner| | 
| itemID | string | Item ID| | 


#### Remarks :
* <span style="color:red;font-size:13px">Token which Modifiable Flag is TRUE, Item owner allowed to modify item-metadata.</span>
* <span style="color:red;font-size:13px">Token which Modifiable Flag is FALSE, only Token owner allowed to modify the item-metadata.</span>


