### MsgTransferNonFungibleTokenOwnership
This is the message type used to transfer-ownership of a non-fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Token owner| | 
| to | string | true   | New token owner| | 


Example

```
{
    "type": "nonFungible/transferNonFungibleTokenOwnership",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgTransferNonFungibleTokenOwnership` message is received.

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
		'case MsgTransferNonFungibleTokenOwnership:
			return handleMsgTransferNonFungibleTokenOwnership(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the `MsgTransferNonFungibleTokenOwnership` message in `handleMsgTransferNonFungibleTokenOwnership`:

```
func (k *Keeper) transferNonFungibleTokenOwnership(ctx sdkTypes.Context, from sdkTypes.AccAddress, to sdkTypes.AccAddress, token *Token) sdkTypes.Result {

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return types.ErrInvalidTokenOwner().Result()
	}

	if ownerWalletAccount != nil && !token.Owner.Equals(from) {
		return types.ErrInvalidTokenOwner().Result()
	}

	if !token.IsApproved() {
		// TODO: Please define an error code
		return sdkTypes.ErrUnknownRequest("Token is not approved.").Result()
	}

	if token.IsFrozen() {
		return types.ErrTokenFrozen().Result()
	}

	// set token newowner to new owner, pending for accepting by new owner
	token.NewOwner = to
	token.Flags.AddFlag(TransferTokenOwnershipFlag)

	k.storeToken(ctx, token.Symbol, token)

	eventParam := []string{token.Symbol, from.String(), to.String()}
	eventSignature := "TransferredNonFungibleTokenOwnership(string,string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.  

* A valid Token.
* Token must be approved, and not yet be freeze.
* Signer must be valid token owner
* Action of Re-transfer-ownership is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{token.Symbol, from.String(), to.String()}
eventSignature := "TransferredNonFungibleTokenOwnership(string,string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using TransferredNonFungibleTokenOwnership(string,string,string)
* from : Token owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| from | string | Token owner| | 
| to | string | New token owner| | 

