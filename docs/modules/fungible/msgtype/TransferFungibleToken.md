This is the message type used to transfer the fungible token.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |
| to | string | true   | New token owner| |
| value | int | true   | Value| |


Example
```
{
    "type": "token/transferFungibleToken",
    "value": {
        "symbol": "TT-6",
        "value": "0",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw1w0m8xqy0fpgkf6luwu666f5hhl3tm0sq53snw5"
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgTransferFungibleToken` message is received.

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
		'case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)'
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
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
Now, you need to define the actual logic for handling the MsgTypeTransferFungibleToken message in `handleMsgTransferFungibleToken`:

```
func (k *Keeper) TransferFungibleToken(ctx sdkTypes.Context, symbol string, from, to sdkTypes.AccAddress, value sdkTypes.Uint) sdkTypes.Result {
	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return types.ErrTokenInvalid().Result()
	}

	fromAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromAccount == nil {
		return types.ErrInvalidTokenAccount().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	ownerAccount := k.getFungibleAccount(ctx, symbol, from)
	if ownerAccount == nil {
		return sdkTypes.ErrUnknownRequest("Owner doesn't have such token").Result()
	}

	if ownerAccount.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	if ownerAccount.Balance.LT(value) {
		return types.ErrInvalidTokenAccountBalance(fmt.Sprintf("Not enough tokens. Have only %v", ownerAccount.Balance.String())).Result()
	}

	newOwnerAccount := k.getFungibleAccount(ctx, symbol, to)
	if newOwnerAccount == nil {
		newOwnerAccount = k.createFungibleAccount(ctx, symbol, to)
	}

	if newOwnerAccount.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	subFungibleTokenErr := k.subFungibleToken(ctx, symbol, from, value)
	if subFungibleTokenErr != nil {
		return subFungibleTokenErr.Result()
	}

	addFungibleTokenErr := k.addFungibleToken(ctx, symbol, to, value)
	if addFungibleTokenErr != nil {
		return addFungibleTokenErr.Result()
	}

	eventParam := []string{symbol, from.String(), to.String(), value.String()}
	eventSignature := "TransferredFungibleToken(string,string,string,bignumber)"

	accountSequence := fromAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Signer must be a valid token owner.
* Current token owner's account is not in freeze condition.
* New token owner's account is not in freeze condition.
* Current token owner must have enough balance in order to do transfer amount to new token owner
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Transfer.
* Action of Re-transfer is allowed if have enough balance.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, from.String(), to.String(), value.String()}
eventSignature := "TransferredFungibleToken(string,string,string,bignumber)"

accountSequence := fromAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using TransferredFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| from | string | Token owner| |
| to | string | New token owner| |
| value | string | Value| |

