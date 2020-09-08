This is the message type used to transfer the item of a non-fungible token.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| | 
| from | string | true   | Item owner| | 
| to | string | true   | New Item owner| | 
| itemID | string | true   | Properties of token| | 


Example
```
{
    "type": "nonFungible/transferNonFungibleItem",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6",
        "to": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
        "itemID": "item-123"
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgTransferNonFungibleItem` message is received.

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
		'case MsgTransferNonFungibleItem:
			return handleMsgTransferNonFungibleItem(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the `MsgTransferNonFungibleItem` message in `handleMsgTransferNonFungibleItem`:

```
func (k *Keeper) TransferNonFungibleItem(ctx sdkTypes.Context, symbol string, from, to sdkTypes.AccAddress, itemID string) sdkTypes.Result {
	if !k.IsItemOwner(ctx, symbol, itemID, from) {
		return types.ErrInvalidItemOwner().Result()
	}

	var token = new(Token)
	if exists := k.GetTokenDataInfo(ctx, symbol, token); !exists {
		return types.ErrTokenInvalid().Result()
	}

	if !token.Flags.HasFlag(TransferableFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	fromAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromAccount == nil {
		return types.ErrInvalidTokenAccount().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	itemKey := getNonFungibleItemKey(symbol, []byte(itemID))
	ownerKey := getNonFungibleItemOwnerKey(symbol, []byte(itemID))

	store := ctx.KVStore(k.key)

	ownerValue := store.Get(ownerKey)
	if ownerValue == nil {
		return types.ErrInvalidTokenOwner().Result()
	}

	itemValue := store.Get(itemKey)
	if itemValue == nil {
		return types.ErrInvalidTokenOwner().Result()
	}

	var item = new(Item)
	k.cdc.MustUnmarshalBinaryLengthPrefixed(itemValue, item)
	if k.IsItemTransferLimitExceeded(ctx, symbol, itemID) {

		// TO-DO: own error message.
		return sdkTypes.ErrInternal("Item has existed transfer limit.").Result()
	}

	// delete old owner
	store.Delete(ownerKey)

	// set to new owner
	store.Set(ownerKey, to.Bytes())

	// increase the transfer limit and set
	item.TransferLimit = item.TransferLimit.Add(sdkTypes.NewUint(1))
	itemData := k.cdc.MustMarshalBinaryLengthPrefixed(item)
	store.Set(itemKey, itemData)

	eventParam := []string{symbol, string(itemID), from.String(), to.String()}
	eventSignature := "TransferredNonFungibleItem(string,string,string,string)"

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
* Token transferable flag equals to true and not in freeze condition.
* Token which Transfer-limit Flag set to a certain threshold, number of items that can be transferrable need to follow this constraint.
* Signer must be a valid item owner.
* Action of Re-transfer is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, string(itemID), from.String(), to.String()}
eventSignature := "TransferredNonFungibleItem(string,string,string,string)"

accountSequence := fromAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using TransferredNonFungibleItem(string,string,string,string)
* from : Item owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| itemID | string | Item ID| | 
| from | string | Item owner| | 
| to | string | New item owner| | 
