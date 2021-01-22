This is the message type used to endorse an item of a non-fungible token.
The user can write into the MEMO field that indicate what kind of remarks/comments wish to be input, while come to item endorsement process. However, the limitation of this MEMO can only be 128 bytes.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Endorser| | 
| itemID | string | true   | Item ID, which must be unique| | 
| metadata | string | true   | metadata of endorsement| | 


Example
```
{
    "type": "nonFungible/endorsement",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1k9sxz0h3yeh0uzmxet2rmsj7xe5zg54eq7vhla",
        "itemID": "ITEM-123",
        "metadata": "metadata for endorsement"
    }
}


```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgEndorsement` message is received.

In the file (./x/token/nonfungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateNonFungibleToken:
			return handleMsgCreateNonFungibleToken(ctx, keeper, msg)
		case MsgSetNonFungibleTokenStatus:
			return handleMsgSetNonFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintNonFungibleItem:
			return handleMsgMintNonFungibleItem(ctx, keeper, msg)
		case MsgTransferNonFungibleItem:
			return handleMsgTransferNonFungibleItem(ctx, keeper, msg)
		case MsgBurnNonFungibleItem:
			return handleMsgBurnNonFungibleItem(ctx, keeper, msg)
		case MsgSetNonFungibleItemStatus:
			return handleMsgSetNonFungibleItemStatus(ctx, keeper, msg)
		case MsgTransferNonFungibleTokenOwnership:
			return handleMsgTransferNonFungibleTokenOwnership(ctx, keeper, msg)
		case MsgAcceptNonFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		'case MsgEndorsement:
			return handleMsgEndorsement(ctx, keeper, msg)'
		case MsgUpdateItemMetadata:
			return handleMsgUpdateItemMetadata(ctx, keeper, msg)
		case MsgUpdateNFTMetadata:
			return handleMsgUpdateNFTMetadata(ctx, keeper, msg)
    case MsgUpdateEndorserList:
			return handleMsgUpdateEndorserList(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible token Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}

}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the `MsgEndorsement` message in `handleMsgEndorsement`:

```
func (k *Keeper) MakeEndorsement(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress, itemID, metadata string) sdkTypes.Result {

	// validation of exisisting owner account
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, from)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid endorser.").Result()
	}

	item := k.GetNonFungibleItem(ctx, symbol, itemID)
	if item == nil {
		return types.ErrTokenInvalid().Result()
	}

	eventParam := []string{symbol, string(itemID), from.String(), metadata}
	eventSignature := "EndorsedNonFungibleItem(string,string,string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}

}


```

In this function, requirements need to be met before emitted by the network.  

* A valid Item ID.
* Signer must be a valid endorser.
* Action of Re-endorse process is allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, string(itemID), from.String(), metadata}
eventSignature := "EndorsedNonFungibleItem(string,string,string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
	Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
	Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using EndorsedNonFungibleItem(string,string,string)
* from : Endorser
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| itemID | string | Item ID| | 
| from | string | Endorser| | 
| metadata | string | metadata| | 

