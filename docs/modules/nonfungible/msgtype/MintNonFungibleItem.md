This is the message type used to mint an item of a non-fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| itemID | string | true   | Item ID, which must be unique| | 
| symbol | string | true   | Token symbol, which must be unique| | 
| owner | string | true   | Token owner| | 
| to | string | true   | Item owner| | 
| properties | string | true   | Properties of item| | 
| metadata | string | true   | Metadata of item| | 



Example
```
{
    "type": "nonFungible/mintNonFungibleItem",
    "value": {
        "itemID": "ITEM-123",
        "symbol": "TNFT",
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
        "properties": "item-properties",
        "metadata": "item-metadata"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgMintNonFungibleItem` message is received.

In the file (./x/token/nonfungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateNonFungibleToken:
			return handleMsgCreateNonFungibleToken(ctx, keeper, msg)
		case MsgSetNonFungibleTokenStatus:
			return handleMsgSetNonFungibleTokenStatus(ctx, keeper, msg)
		'case MsgMintNonFungibleItem:
			return handleMsgMintNonFungibleItem(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the `MsgMintNonFungibleItem` message in `handleMsgMintNonFungibleItem`:

```
func (k *Keeper) MintNonFungibleItem(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress, to sdkTypes.AccAddress, itemID, properties, metadata string) sdkTypes.Result {

	nonFungibleToken := new(Token)

	if exists := k.GetTokenDataInfo(ctx, symbol, nonFungibleToken); !exists {
		return types.ErrInvalidTokenSymbol(symbol).Result()
	}

	// get minter account.
	// if token is public that means minter can be anyone.
	minterAccount := k.accountKeeper.GetAccount(ctx, from)
	if minterAccount == nil {
		return types.ErrInvalidTokenAccount().Result()
	}

	if !nonFungibleToken.Flags.HasFlag(PubFlag) {
		if !nonFungibleToken.Owner.Equals(from) {
			return types.ErrInvalidTokenMinter().Result()
		}
	} else {
		if !from.Equals(to) {
			return sdkTypes.ErrInternal("Public token can only be minted to oneself.").Result()
		}
	}

	if !nonFungibleToken.Flags.HasFlag(MintFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	if !nonFungibleToken.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	if nonFungibleToken.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	if !k.IsItemIDUnique(ctx, nonFungibleToken.Symbol, itemID) {
		return types.ErrTokenItemIDInUsed().Result()
	}

	amt := sdkTypes.NewUint(1)
	nonFungibleToken.TotalSupply = nonFungibleToken.TotalSupply.Add(amt)

	// check mint limit, if token mint limit !=0
	if !nonFungibleToken.MintLimit.IsZero() {

		if k.IsMintLimitExceeded(ctx, nonFungibleToken.Symbol, to) {
			return sdkTypes.ErrInternal("Holding limit existed.").Result()
		}
		k.increaseMintItemLimit(ctx, symbol, to)
	}

	k.storeToken(ctx, symbol, nonFungibleToken)

	item := k.createNonFungibleItem(ctx, nonFungibleToken.Symbol, to, itemID, properties, metadata)

	eventParam := []string{symbol, string(item.ID), from.String(), to.String()}
	eventSignature := "MintedNonFungibleItem(string,string,string,string)"

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
* Token must be approved, and not yet be freeze.
* Token which Public Flag equals to true can only be minted to same user.
* Token which Mint-limit Flag set to ZERO, any items can be minted without the limitation. Otherwise, will base on the threshold of this setting.
* A valid Item ID which must be unique.
* Action of Re-mint is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, string(item.ID), from.String(), to.String()}
eventSignature := "MintedNonFungibleItem(string,string,string,string)"

accountSequence := minterAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using MintedNonFungibleItem(string,string,string,string)
* from : Token owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| itemID | string | Item ID| | 
| from | string | Token owner| | 
| to | string | Item owner| | 
