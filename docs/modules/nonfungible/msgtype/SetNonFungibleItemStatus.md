This is the message type used to update the status of an item of a non-fungible token, 
  eg. Freeze or unfreeze

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Item owner| | 
| payload | ItemPayload | true   | Item Payload information| | 
| signatures | []Signature | true   | Array of Signature| | 


Item Payload Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| item | ItemDetails | true   | Item details information| | 
| pub_key | type/value | true   | crypto.PubKey| | 
| signature | []byte | true   | signature| | 


Item Details Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Item owner| | 
| nonce | string | true   | nonce signature| | 
| status | string | true   | There are two type of status, which include `FREEZE_ITEM`, `UNFREEZE_ITEM`. All this keywords must be matched while come to this message type | | 
| symbol | string | true   | Token-symbol| | 
| itemID | string | true   | Item ID| | 

`status`

| status | Details                 |
| -------- | --------------------------- |
| FREEZE_ITEM  | A valid token with a valid Item ID which already been approved and yet to be frozen must be signed by authorised KYC Signer with valid signature will be proceed, this also along with the approval from authorised address of issuer and provider parties. Item ID which already been frozen is not allowed to do re-submit.| | 
| UNFREEZE_ITEM  | A valid token with a valid Item ID which already been approved and frozen must be signed by authorised KYC Signer with valid signature will be proceed, this also along with the approval from authorised address of issuer and provider parties. Item ID which already been unfreeze is not allowed to do re-submit.| | 

Example
```
{
  "type": "nonFungible/setNonFungibleItemStatus",
  "value": {
    "owner": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
    "payload": {
      "item": {
        "from": "mxw1f8r0k5p7s85kv7jatwvmpartyy2j0s20y0p0yk",
        "nonce": "0",
        "status": "FREEZE_ITEM",
        "symbol": "TNFT",
        "itemID": "Item-123"
      },
      "pub_key": {
        "type": "tendermint/PubKeySecp256k1",
        "value": "A0VBHXKgUEU2fttqh8Lhqp1G6+GzOxTXvCExzDLEdfD7"
      },
      "signature": "hEnOeX536dduNFgWMOdK0cFKq2xwWW9aAHalW9l5kAgxg94P55MIEQJ8vFkEq9eAcYo1sjQ4TfXW5EynIk/kuQ=="
    },
    "signatures": [
      {
        "pub_key": {
          "type": "tendermint/PubKeySecp256k1",
          "value": "Ausyj7Gas2WkCjUpM8UasCcezXrzTMTRbPHqYx44GzLm"
        },
        "signature": "uDh1fll2eN/3lpKgMydW9mEdgfI3Mint30pkFrrQ+phkIY5X8nMbCoS6WNmw5bqCiVunMhkey75U3Qm99csG4g=="
      }
    ]
  }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgSetNonFungibleItemStatus` message is received.

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
		'case MsgSetNonFungibleItemStatus:
			return handleMsgSetNonFungibleItemStatus(ctx, keeper, msg)'
		case MsgTransferNonFungibleTokenOwnership:
			return handleMsgTransferNonFungibleTokenOwnership(ctx, keeper, msg)
		case MsgAcceptNonFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		case MsgEndorsement:
			return handleMsgEndorsement(ctx, keeper, msg)
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
Now, you define the actual logic for handling the `MsgSetNonFungibleItemStatus` - FreezeNonFungibleItem message in `handleMsgSetNonFungibleItemStatus`:

```
func (k *Keeper) FreezeNonFungibleItem(ctx sdkTypes.Context, symbol string, owner, itemOwner sdkTypes.AccAddress, itemID string, metadata string) sdkTypes.Result {
	var token = new(Token)
	if exists := k.GetTokenDataInfo(ctx, symbol, token); !exists {
		return sdkTypes.ErrUnknownRequest("No such non fungible token.").Result()
	}

	if !k.IsAuthorised(ctx, owner) {
		return sdkTypes.ErrUnauthorized("Not authorised to freeze non fungible item.").Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	nonFungibleItem := k.GetNonFungibleItem(ctx, symbol, itemID)
	if nonFungibleItem == nil {
		return sdkTypes.ErrUnknownRequest("No such item to freeze.").Result()
	}

	itemOwner = k.GetNonFungibleItemOwnerInfo(ctx, symbol, itemID)
	if itemOwner == nil {
		return sdkTypes.ErrUnknownRequest("Invalid item owner.").Result()
	}

	if nonFungibleItem.Frozen {
		return sdkTypes.ErrUnknownRequest("Non Fungible item already frozen.").Result()
	}

	nonFungibleItem.Frozen = true

	k.storeNonFungibleItem(ctx, symbol, itemOwner, nonFungibleItem)

	eventParam := []string{symbol, itemID, owner.String()}
	eventSignature := "FrozenNonFungibleItem(string,string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.  

* A valid Token.
* A valid Item ID which must not be freeze.
* Signer must be KYC authorised.
* Action of Re-freeze-item is not allowed.

Next, you define the actual logic for handling the MsgSetNonFungibleItemStatus-UnfreezeNonFungibleItem message in handleMsgSetNonFungibleItemStatus:

```
func (k *Keeper) UnfreezeNonFungibleItem(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, itemID string, metadata string) sdkTypes.Result {
	if !k.IsAuthorised(ctx, owner) {
		return sdkTypes.ErrUnauthorized("Not authorised to unfreeze token account.").Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	if exists := k.GetTokenDataInfo(ctx, symbol, token); !exists {
		return sdkTypes.ErrUnknownRequest("No such non fungible token.").Result()
	}

	nonFungibleItem := k.GetNonFungibleItem(ctx, symbol, itemID)
	if nonFungibleItem == nil {
		return sdkTypes.ErrUnknownRequest("No such  non fungible item to unfreeze.").Result()
	}

	itemOwner := k.GetNonFungibleItemOwnerInfo(ctx, symbol, itemID)
	if itemOwner == nil {
		return sdkTypes.ErrUnknownRequest("Invalid item owner.").Result()
	}

	if !nonFungibleItem.Frozen {
		return sdkTypes.ErrUnknownRequest("Non fungible item not frozen.").Result()
	}

	nonFungibleItem.Frozen = false

	k.storeNonFungibleItem(ctx, symbol, itemOwner, nonFungibleItem)

	eventParam := []string{symbol, itemID, owner.String()}
	eventSignature := "UnfreezeNonFungibleItem(string,string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.  

* A valid Token.
* A valid Item ID which must be freeze.
* Signer must be KYC authorised.
* Action of Re-unfreeze-item is not allowed.


`Events`

1.This tutorial describes how to create maxonrow events for scanner base on freeze-item after emitted by a network.

```
eventParam := []string{symbol, itemID, owner.String()}
eventSignature := "FrozenNonFungibleItem(string,string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using FrozenNonFungibleItem(string,string,string) 
* owner : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| itemID | string | Item ID| | 
| owner | string | Signer| | 


2.This tutorial describes how to create maxonrow events for scanner base on unfreeze-item after emitted by a network.

```
eventParam := []string{symbol, itemID, owner.String()}
eventSignature := "UnfreezeNonFungibleItem(string,string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using UnfreezeNonFungibleItem(string,string,string) 
* owner : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| itemID | string | Item ID| | 
| owner | string | Signer| | 
