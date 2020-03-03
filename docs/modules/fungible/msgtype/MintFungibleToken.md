This is the message type used to mint an item of a non-fungible token.

## Parameters

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| owner | string | true   | Token owner| | 
| to | string | true   | Item owner| | 
| value | int | true   | value| | 
| metadata | string | true   | Metadata of item| | 



#### Example
```
{
    "type": "token/mintFungibleToken",
    "value": {
        "symbol": "TT-2b",
        "value": "100000",
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}

```

## Handler

The role of the handler is to define what action(s) needs to be taken when this MsgTypeMintFungibleToken message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

![Image-1](../pic/AcceptFungibleTokenOwnership_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeMintFungibleToken message in handleMsgMintFungibleToken:

![Image-2](../pic/MintFungibleToken_02.png)


In this function, requirements need to be met before emitted by the network.  

* xxA valid Token.
* xxToken must be approved, and not yet be freeze.
* xxToken which Public Flag equals to true can only be minted to same user.
* xxToken which Mint-limit Flag set to ZERO, any items can be minted without the limitation. Otherwise, will base on the threshold of this setting.
* xxA valid Item ID which must be unique.
* xxAction of Re-mint is not allowed.


## Events
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-3](../pic/MintFungibleToken_03.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using MintedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 
| to | string | Item owner| | 
| value | string | value| | 
