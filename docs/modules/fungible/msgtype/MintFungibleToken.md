This is the message type used to mint the amount of fungible token to an account owner.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| owner | string | true   | Token owner| |
| to | string | true   | Token account owner| |
| value | int | true   | Mint value| |


Example
```
{
    "type": "token/mintFungibleToken",
    "value": {
        "symbol": "TT-2b",
        "value": "100000",
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgMintFungibleToken` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		'case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)'
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
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
Now, you need to define the actual logic for handling the MsgTypeMintFungibleToken message in `handleMsgMintFungibleToken`:

```
func (k *Keeper) MintFungibleToken(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress, to sdkTypes.AccAddress, value sdkTypes.Uint) sdkTypes.Result {

	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return types.ErrInvalidTokenSymbol(symbol).Result()
	}

	if !token.Flags.HasFlag(MintFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	// Wallet account
	minterAccount := k.accountKeeper.GetAccount(ctx, from)
	if minterAccount == nil {
		return types.ErrInvalidTokenMinter().Result()
	}

	// minter can only be the owner of the token
	tokenOwnerAccount := k.getFungibleAccount(ctx, symbol, from)

	// token account
	if tokenOwnerAccount == nil {
		return types.ErrInvalidTokenMinter().Result()
	}

	if tokenOwnerAccount.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	if !token.Owner.Equals(from) {
		return types.ErrInvalidTokenMinter().Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	account := k.getFungibleAccount(ctx, symbol, to)
	if account == nil {
		account = k.createFungibleAccount(ctx, symbol, to)
	}

	if account.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	token.TotalSupply = token.TotalSupply.Add(value)

	// max supply 0 means is dynamic supply
	if !token.MaxSupply.IsZero() {
		if token.TotalSupply.GT(token.MaxSupply) {
			return types.ErrInvalidTokenSupply().Result()
		}
	}

	addFungibleTokenErr := k.addFungibleToken(ctx, symbol, to, value)
	if addFungibleTokenErr != nil {
		return addFungibleTokenErr.Result()
	}

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, "mxw000000000000000000000000000000000000000", to.String(), value.String()}
	eventSignature := "MintedFungibleToken(string,string,string,bignumber)"

	accountSequence := minterAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token which Mint Flag equals to true and must be approved, and not yet be freeze.
* Token can only be minted by token-creator.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer, Burn, Freeze.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to Transfer, Burn, Freeze and Transfer-ownership.
* Action of Re-mint is allowed if not yet reached the total-supply-limit (as Fixed-Supply).


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, "mxw000000000000000000000000000000000000000", to.String(), value.String()}
eventSignature := "MintedFungibleToken(string,string,string,bignumber)"

accountSequence := minterAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using MintedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token account owner| |
| to | string | Token account owner| |
| value | string | Mint value| |

