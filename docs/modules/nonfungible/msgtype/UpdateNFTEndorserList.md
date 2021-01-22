This is the message type used to update Endorser list of a non-fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Token owner| | 
| endorsers | array of string | true   | Endorser list| | 

Example
```
{
  "type":"nonFungible/updateEndorserList",
  "value":{
    "symbol":"TNFT",
    "from":"mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
    "endorsers":[
      "mxw1f8r0k5p7s85kv7jatwvmpartyy2j0s20y0p0yk",
      "mxw1k9sxz0h3yeh0uzmxet2rmsj7xe5zg54eq7vhla"
    ]
  }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgUpdateNFTEndorserList` message is received.

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
		case MsgEndorsement:
			return handleMsgEndorsement(ctx, keeper, msg)
		case MsgUpdateItemMetadata:
			return handleMsgUpdateItemMetadata(ctx, keeper, msg)
		case MsgUpdateNFTMetadata:
			return handleMsgUpdateNFTMetadata(ctx, keeper, msg)
    'case MsgUpdateEndorserList:
			return handleMsgUpdateEndorserList(ctx, keeper, msg)'
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible token Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}

}
```

NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the `MsgUpdateEndorserList` message in `handleMsgUpdateEndorserList`:

```
func (k *Keeper) UpdateNFTEndorserList(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress, endorsers []sdkTypes.AccAddress) sdkTypes.Result {

	// validation of exisisting owner account
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, from)
	if ownerWalletAccount == nil {
		return types.ErrInvalidTokenOwner().Result()
	}

	var token = new(Token)

	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	if !token.Owner.Equals(from) {
		return types.ErrInvalidTokenOwner().Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	token.EndorserList = endorsers
	k.storeToken(ctx, symbol, token)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	eventParam := []string{symbol, from.String()}
	eventSignature := "UpdatedNonFungibleTokenEndorserList(string,string)"

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.  

* A valid Token which been approved.
* Token not in freeze condition.
* Signer must be valid token owner.
* Action of Re-update is allowed.

`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

eventParam := []string{symbol, from.String()}
eventSignature := "UpdatedNonFungibleTokenEndorserList(string,string)"

return sdkTypes.Result{
  Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
  Log:    resultLog.String(),
}

```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using UpdatedNonFungibleTokenEndorserList(string,string)
* from : Token owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| from | string | Token owner| | 

