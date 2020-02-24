# MsgTypeBurnNonFungibleItem

This is the message type used to burn an item of a non-fungible token.


#### Parameters
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Item owner| | 
| itemID | string | true   | Properties of token| | 



#### Example

```
{
    "type": "nonFungible/burnNonFungibleItem",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6",
        "itemID": "ITEM-123"
    }
}

```

#### Listening to events of MsgTypeBurnNonFungibleItem
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](/en/latest/pic_module/MsgTypeBurnNonFungibleItem.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using BurnedNonFungibleItem(string,string,string)
* Item owner
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Item owner| | 
| itemID | string | Item ID| | 


