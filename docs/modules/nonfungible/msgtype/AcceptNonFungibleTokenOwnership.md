This is the message type used to accept-ownership of a non-fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Token owner| | 


Example

```
{
    "type": "nonFungible/acceptNonFungibleTokenOwnership",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6"
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgAcceptNonFungibleTokenOwnership` message is received.

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
		'case MsgAcceptNonFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the `MsgAcceptNonFungibleTokenOwnership` message in `handleMsgAcceptNonFungibleTokenOwnership`:

```
func (k *Keeper) acceptNonFungibleTokenOwnership(ctx sdkTypes.Context, from sdkTypes.AccAddress, token *Token) sdkTypes.Result {

	if !token.Flags.HasFlag(AcceptTokenOwnershipFlag) && !token.Flags.HasFlag(ApproveTransferTokenOwnershipFlag) && !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	// validation of exisisting owner account
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	// validation of new owner account
	newOwnerWalletAccount := k.accountKeeper.GetAccount(ctx, token.NewOwner)
	if newOwnerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid new token owner.").Result()
	}

	if newOwnerWalletAccount != nil && token.NewOwner.String() != from.String() {
		return types.ErrInvalidTokenNewOwner().Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return sdkTypes.ErrUnknownRequest("Token is not approved.").Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	//TO-DO: if there is need to set token.NewOwner to empty
	// accepting token ownership, remove newowner move the newowner into owner.
	var emptyAccAddr sdkTypes.AccAddress
	token.Owner = from
	token.NewOwner = emptyAccAddr

	token.Flags.RemoveFlag(ApproveTransferTokenOwnershipFlag)
	token.Flags.RemoveFlag(AcceptTokenOwnershipFlag)
	token.Flags.RemoveFlag(TransferTokenOwnershipFlag)
	k.storeToken(ctx, token.Symbol, token)

	eventParam := []string{token.Symbol, from.String()}
	eventSignature := "AcceptedNonFungibleTokenOwnership(string,string)"

	accountSequence := newOwnerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.  

* A valid Token.
* A valid Token owner.
* A valid New token owner.
* Token must be approved and not be freeze.
* Action of Re-accept-ownership is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{token.Symbol, from.String()}
eventSignature := "AcceptedNonFungibleTokenOwnership(string,string)"

accountSequence := newOwnerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}

```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using AcceptedNonFungibleTokenOwnership(string,string)
* from : Token owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| from | string | Token owner| | 
