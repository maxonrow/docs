This is the message type used to burn an item of a non-fungible token.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Item owner| | 
| itemID | string | true   | Properties of token| | 



Example

```
{
    "type": "nonFungible/burnNonFungibleItem",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6",
        "itemID": "ITEM-123"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgBurnNonFungibleItem` message is received.

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
		'case MsgBurnNonFungibleItem:
			return handleMsgBurnNonFungibleItem(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the `MsgBurnNonFungibleItem` message in `handleMsgBurnNonFungibleItem`:

```
func (k *Keeper) BurnNonFungibleItem(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress, itemID string) sdkTypes.Result {
	var token = new(Token)
	if exists := k.GetTokenDataInfo(ctx, symbol, token); !exists {
		return types.ErrInvalidTokenSymbol(symbol).Result()
	}

	if !token.Flags.HasFlag(BurnFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	fromAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid account to burn from.").Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	item := k.GetNonFungibleItem(ctx, symbol, itemID)
	if item == nil {
		return types.ErrTokenItemNotFound().Result()
	}

	itemOwner := k.GetNonFungibleItemOwnerInfo(ctx, symbol, itemID)
	if !itemOwner.Equals(from) {
		return types.ErrInvalidTokenOwner().Result()
	}

	ownerKey := getNonFungibleItemOwnerKey(symbol, []byte(itemID))
	itemKey := getNonFungibleItemKey(symbol, []byte(itemID))

	store := ctx.KVStore(k.key)

	store.Delete(itemKey)
	store.Delete(ownerKey)

	eventParam := []string{symbol, string(item.ID), from.String()}
	eventSignature := "BurnedNonFungibleItem(string,string,string)"

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
* Token must be approved before this, and not be freeze. Also burnable flag must equals to true.
* A valid Item ID.
* Signer who is the Item owner need to be authorised to do this process.
* Action of Re-burn is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, string(item.ID), from.String()}
eventSignature := "BurnedNonFungibleItem(string,string,string)"

accountSequence := fromAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using BurnedNonFungibleItem(string,string,string)
* from : Item owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| itemID | string | Item ID| | 
| from | string | Item owner| | 
