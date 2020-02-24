# MsgTypeMintNonFungibleItem

This is the message type used to mint an item of a non-fungible token.


#### Parameters
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| itemID | string | true   | Item ID, which must be unique| | 
| symbol | string | true   | Token symbol, which must be unique| | 
| owner | string | true   | Token owner| | 
| to | string | true   | Item owner| | 
| properties | string | true   | Properties of item| | 
| metadata | string | true   | Metadata of item| | 



#### Example
```
{
    "type": "nonFungible/mintNonFungibleItem",
    "value": {
        "itemID": "ITEM-123",
        "symbol": "TNFT",
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
        "properties": "item-properties",
        "metadata": "item-metadata"
    }
}

```

#### Listening to events of MsgTypeMintNonFungibleItem
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](/en/latest/pic_module/MsgTypeMintNonFungibleItem.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using MintedNonFungibleItem(string,string,string,string)
* Token owner
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 
| to | string | Item owner| | 
| itemID | string | Item ID| | 


#### Remarks :
* <span style="color:red;font-size:13px">Token which Public Flag is TRUE can only be minted to same user.</span>
* <span style="color:red;font-size:13px">Token which Mint-limit Flag set to ZERO, any items can be minted without the limitation. Otherwise, will base on the threshold of this setting.</span> 

