This is the message type used to do KYC whitelisting (by Bussiness users).


## Parameters

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| value | string | true   | value| | 
| itemID | string | true   | Properties of token| | 


#### Example

```
{
    "type": "kyc/kycBind",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6",
        "itemID": "ITEM-123"
    }
}

```

## Handler

The role of the handler is to define what action(s) needs to be taken when this MsgKycBind message is received.

In the file (./x/kyc/handler.go) start with the following code:

![Image-1](../pic/MintNonFungibleItem_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgKycBind message in handleMsgBurnNonFungibleItem:

![Image-2](../pic/KycBind_02.png)


In this function, requirements need to be met before emitted by the network.  

* A valid Token.
* Token must be approved before this, and not be freeze. Also burnable flag must equals to true.
* A valid Item ID.
* Signer who is the Item owner need to be authorised to do this process.
* Action of Re-burn is not allowed.


## Events
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](../pic/KycBind_03.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using xxBurnedFungibleToken(string,string,string,bignumber)
* Item owner
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Item owner| | 
| account | string | Account address| | 
| value | string | value| | 

