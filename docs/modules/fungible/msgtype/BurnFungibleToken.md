This is the message type used to burn the amount of fungible token of an account owner.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| value | string | true   | Burn value| |
| from | string | true   | Token account owner| |


Example
```
{
    "type": "token/burnFungibleToken",
    "value": {
        "symbol": "TT-2",
        "value": "200",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgBurnFungibleToken` message is received.

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
		'case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the MsgTypeBurnFungibleToken message in `handleMsgBurnFungibleItem`:

```
func (k *Keeper) BurnFungibleToken(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, value sdkTypes.Uint) sdkTypes.Result {
	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return types.ErrInvalidTokenSymbol(symbol).Result()
	}

	if !token.Flags.HasFlag(BurnFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	ownerAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid account to burn from.").Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	account := k.getFungibleAccount(ctx, symbol, owner)
	if account == nil {
		return types.ErrInvalidTokenAccount().Result()
	}

	if account.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	if account.Balance.LT(value) {
		return types.ErrInvalidTokenAccountBalance(fmt.Sprintf("Not enough tokens. Have only %v", account.Balance.String())).Result()
	}

	token.TotalSupply = token.TotalSupply.Sub(value)
	k.storeToken(ctx, symbol, token)

	subFungibleTokenErr := k.subFungibleToken(ctx, symbol, owner, value)
	if subFungibleTokenErr != nil {
		return subFungibleTokenErr.Result()
	}

	eventParam := []string{symbol, owner.String(), "mxw000000000000000000000000000000000000000", value.String()}
	eventSignature := "BurnedFungibleToken(string,string,string,bignumber)"

	accountSequence := ownerAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token account.
* Token must be approved before this, and not be freeze. Also burnable flag must equals to true.
* Signer who is the Token owner need to be authorised to do this process.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Burn.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Burn.
* Action of Re-burn is allowed if the balance amount is enough to do burning.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, owner.String(), "mxw000000000000000000000000000000000000000", value.String()}
eventSignature := "BurnedFungibleToken(string,string,string,bignumber)"

accountSequence := ownerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using BurnedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token account owner| |
| account | string | Token account owner| |
| value | string | Burn value| |

