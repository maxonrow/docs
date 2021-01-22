This is the message type used to create a non-fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| name | string | true   | Token name| | 
| symbol | string | true   | Token symbol, which must be unique| | 
| properties | string | true   | Properties of token| | 
| metadata | string | true   | Metadata of token| | 
| owner | string | true   | Token owner| | 
| fee | Fee | true   | Application Fee information| | 


Application Fee Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| to | string | true   | Fee-collector| | 
| value | string | true   | Fee amount to be paid| | 


Example
```
{
    "type": "nonFungible/createNonFungibleToken",
    "value": {
        "name": "TestNonFungibleToken",
        "symbol": "TNFT",
        "properties": "token properties",
        "metadata": "token metadata",
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "fee": {
            "to": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
            "value": "10000000"
        }
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgCreateNonFungibleToken` message is received.

In the file (./x/token/nonfungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		'case MsgCreateNonFungibleToken:
			return handleMsgCreateNonFungibleToken(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the `MsgCreateNonFungibleToken` message in `handleMsgCreateNonFungibleToken`:

```
func (k *Keeper) CreateNonFungibleToken(ctx sdkTypes.Context, name string, symbol string, owner sdkTypes.AccAddress, properties string, metadata string, fee Fee) sdkTypes.Result {

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Not authorised to apply for token creation.").Result()
	}

	if k.TokenExists(ctx, symbol) {
		return types.ErrTokenExists(symbol).Result()
	}

	// Overwrite the cosmos sdk tags.
	// Application fee paid at ante.go, event created here.
	amt, parseErr := sdkTypes.ParseCoins(fee.Value + types.CIN)
	if parseErr != nil {
		return sdkTypes.ErrInvalidCoins("Parse value to coins failed.").Result()
	}
	applicationFeeResult := bank.MakeBankSendEvent(ctx, owner, fee.To, amt, *k.accountKeeper)

	zero := sdkTypes.NewUintFromString("0")

	token := &Token{
		Name:        name,
		Flags:       NonFungibleTokenMask,
		Symbol:      symbol,
		Owner:       owner,
		Properties:  properties,
		Metadata:    metadata,
		TotalSupply: zero,
	}

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, owner.String(), fee.To.String(), fee.Value}
	eventSignature := "CreatedNonFungibleToken(string,string,string,bignumber)"
	event := types.MakeMxwEvents(eventSignature, owner.String(), eventParam)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())
	return sdkTypes.Result{
		Events: applicationFeeResult.Events.AppendEvents(event),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.  

* Token must be unique and not existed.
* Token owner must be KYC authorised.
* A valid Fee will be charged base on this.
* Action of Re-create is not allowed.

`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
	eventParam := []string{symbol, owner.String(), fee.To.String(), fee.Value}
	eventSignature := "CreatedNonFungibleToken(string,string,string,bignumber)"
	event := types.MakeMxwEvents(eventSignature, owner.String(), eventParam)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())
	return sdkTypes.Result{
		Events: applicationFeeResult.Events.AppendEvents(event),
		Log:    resultLog.String(),
	}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using CreatedNonFungibleToken(string,string,string,bignumber)
* from : Token owner
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 
| to | string | Fee-collector| | 
| value | bignumber | Fee amount to be paid| | 
