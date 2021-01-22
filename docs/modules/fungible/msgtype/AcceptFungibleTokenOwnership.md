This is the message type used to accept the ownership of a fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |


Example

```
{
    "type": "token/acceptFungibleTokenOwnership",
    "value": {
        "symbol": "TT-6",
        "from": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgAcceptFungibleTokenOwnership` message is received.

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
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		'case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)'
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeAcceptFungibleTokenOwnership message in `handleMsgTypeAcceptFungibleTokenOwnership`:

```
func (k *Keeper) acceptFungibleTokenOwnership(ctx sdkTypes.Context, from sdkTypes.AccAddress, token *Token, metadata string) sdkTypes.Result {

	if !token.Flags.HasFlag(AcceptTokenOwnershipFlag) && !token.Flags.HasFlag(ApproveTransferTokenOwnershipFlag) && !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	// validation of exisisting owner account
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	// validation of new owner account
	newOwnerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if newOwnerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	newOwnerAccount := k.getFungibleAccount(ctx, token.Symbol, from)
	if newOwnerAccount == nil {
		return sdkTypes.ErrUnknownRequest("New owner account is not found.").Result()
	}

	if newOwnerAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("New owner is frozen").Result()
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
	token.Metadata = metadata

	token.Flags.RemoveFlag(ApproveTransferTokenOwnershipFlag)
	token.Flags.RemoveFlag(AcceptTokenOwnershipFlag)
	token.Flags.RemoveFlag(TransferTokenOwnershipFlag)
	k.storeToken(ctx, token.Symbol, token)

	eventParam := []string{token.Symbol, from.String()}
	eventSignature := "AcceptedFungibleTokenOwnership(string,string)"

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
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer-ownership.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Transfer-ownership.
* Action of Re-accept-ownership is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{token.Symbol, from.String()}
eventSignature := "AcceptedFungibleTokenOwnership(string,string)"

accountSequence := newOwnerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using AcceptedFungibleTokenOwnership(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| from | string | New token owner| |

