This is the message type used to create the fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| name | string | true   | Token name| |
| symbol | string | true   | Token symbol, which must be unique| |
| decimals | int | true   | Value of decimal places for token| |
| metadata | string | true   | Metadata of token| |
| fixedSupply | bool | true   | Fixed Supply value| |
| owner | string | true   | Token owner| |
| maxSupply | int | true   | Maximum Supply value| |
| fee | Fee | true   | Fee information| |

`fixedSupply`

* For the fixedSupply equals `TRUE`, token NOT allowed for minting. The limit of the maxSupply can be ZERO or greater than ZERO. 
* For the fixedSupply equals `FALSE`, means this is dynamic supply. This allowed to mint with any amount of token but not allowed more than the maxSupply limit.
* Upon the process of approval, User NOT allowed to update this setting again.

`maxSupply`

* User must input the number as numeric type during this process, which the value can be set as ZERO or greater than ZERO.
* For dynamic supply, the amount of token be minted can not exceed the maxSupply limit. 
* Upon the process of approval, User NOT allowed to update this setting again.



Fee Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| to | string | true   | Fee-collector| |
| value | string | true   | Fee amount to be paid| |


Example
```
{
    "type": "token/createFungibleToken",
    "value": {
        "name": "TestToken-6",
        "symbol": "TT-6",
        "decimals": "8",
        "metadata": "",
        "fixedSupply": false,
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "maxSupply": "100000",
        "fee": {
            "to": "mxw1g6cjz0pgtchedjyacjcsldhmxcvu2z4nrud9qt",
            "value": "100000"
        }
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgCreateFungibleToken`
 message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		'case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)'
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
Now, you need to define the actual logic for handling the MsgTypeCreateFungibleToken message in `handleMsgCreateFungibleToken`:

```
func (k *Keeper) CreateFungibleToken(ctx sdkTypes.Context, name string, symbol string, decimals int, owner sdkTypes.AccAddress, fixedSupply bool, maxSupply sdkTypes.Uint, metadata string, fee Fee,
) sdkTypes.Result {

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
		Flags:       FungibleFlag,
		Symbol:      symbol,
		Decimals:    decimals,
		Owner:       owner,
		Metadata:    metadata,
		MaxSupply:   maxSupply,
		TotalSupply: zero,
	}

	if !fixedSupply {
		token.Flags.AddFlag(MintFlag)
	}

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, owner.String(), fee.To.String(), fee.Value}
	eventSignature := "CreatedFungibleToken(string,string,string,bignumber)"
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

* Token must be unique.
* Token owner must be KYC authorised.
* A valid Fee will be charged base on this.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer, Burn, Freeze and Transfer-ownership.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached.
* Action of Re-create is not allowed.

`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, owner.String(), fee.To.String(), fee.Value}
eventSignature := "CreatedFungibleToken(string,string,string,bignumber)"
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

* Custom Event Signature : using CreatedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| to | string | Fee-collector| |
| value | bignumber | Fee amount to be paid| |

