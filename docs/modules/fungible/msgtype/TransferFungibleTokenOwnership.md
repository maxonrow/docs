This is the message type used to transfer the ownership of a fungible token.


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
    "type": "token/transferFungibleTokenOwnership",
    "value": {
        "symbol": "TT-6",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgTransferFungibleTokenOwnership` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		'case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)'
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeTransferFungibleTokenOwnership message in `handleMsgTransferFungibleTokenOwnership`:

```
func (k *Keeper) transferFungibleTokenOwnership(ctx sdkTypes.Context, from sdkTypes.AccAddress, to sdkTypes.AccAddress, token *Token, metadata string) sdkTypes.Result {

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

	newOwnerAccount := k.getFungibleAccount(ctx, token.Symbol, to)
	if newOwnerAccount == nil {
		newOwnerAccount = k.createFungibleAccount(ctx, token.Symbol, to)
	}

	if newOwnerAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("New owner is frozen").Result()
	}

	// set token newowner to new owner, pending for accepting by new owner
	token.NewOwner = to
	token.Metadata = metadata
	token.Flags.AddFlag(TransferTokenOwnershipFlag)

	k.storeToken(ctx, token.Symbol, token)

	eventParam := []string{token.Symbol, from.String(), to.String()}
	eventSignature := "TransferredFungibleTokenOwnership(string,string,string)"

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
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer-ownership.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Transfer-ownership.
* Action of Re-transfer-ownership is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{token.Symbol, from.String(), to.String()}
eventSignature := "TransferredFungibleTokenOwnership(string,string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using TransferredFungibleTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |
