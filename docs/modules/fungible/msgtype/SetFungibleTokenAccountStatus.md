This is the message type used to set the account status of a fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Item owner| |
| payload | TokenAccountPayload | true   | Account Payload information| |
| signatures | []Signature | true   | Array of Signature| |


Account Payload Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| tokenAccount | TokenAccount | true   | Token account| |
| pub_key | nil | true   | crypto.PubKey| |
| signature | []byte | true   | Signature| |


Token Account Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Token owner| |
| nonce | string | true   | nonce signature| |
| status | string | true   | There are different type of status, which include FREEZE_ACCOUNT, UNFREEZE_ACCOUNT. All this keywords must be matched while come to this message type | |
| symbol | string | true   | Token-symbol| |
| to | string | true   | Token account address| |

`status`

| status | Details                 |
| -------- | --------------------------- |
| FREEZE_ACCOUNT  | A valid token which already been approved must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been frozen is not allowed to do re-submit.| | 
| UNFREEZE_ACCOUNT  | A valid token which already been approved and frozen must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been unfreeze is not allowed to do re-submit.| | 


Example
```
{
    "type": "token/setFungibleTokenAccountStatus",
    "value": {
        "owner": "mxw1j4duwuaqdj2na054rmlg3pdzncpmwzdjwtfqht",
        "payload": {
            "tokenAccount": {
                "from": "mxw1lmym5599yja76d2s463390he22pcpng3zzpx4p",
                "nonce": "0",
                "status": "FREEZE_ACCOUNT",
                "symbol": "TFT-Acc1",
                "to": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73"
            },
            "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A8I4AHjOPpkQhEFy24L8SZkwd9UxQyEyudZYcYl9f4jK"
            },
            "signature": "XgDB/IWOX2M2gwi+1Voi4YaNhCJzAZdG00hbUTtTAFhEtnl+7i1BHi70XX5pwH9s+74y5gZunT2BFZoJXm/xzA=="
        },
        "signatures": [
            {
                "pub_key": {
                    "type": "tendermint/PubKeySecp256k1",
                    "value": "A4PwoBS8fl/2W+V1HXrWQBn5jXci5gnYUTQzDZFqD0vl"
                },
                "signature": "STAuPiC0512i3HHYm6pK+k0LXwCMLr1DgAjO+bV3UAct068WnvoWNh1Jd7xcWrnaghLfdWZT8yknhEkKd9XFRA=="
            }
        ]
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgSetFungibleTokenAccountStatus` message is received.

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
		'case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)'
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

First, you define the actual logic for handling the MsgTypeSetFungibleTokenAccountStatus-FreezeTokenAccount message in `handleMsgSetFungibleTokenAccountStatus`:

```
func (k *Keeper) FreezeFungibleTokenAccount(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, tokenAccount sdkTypes.AccAddress, metadata string) sdkTypes.Result {
	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return sdkTypes.ErrUnknownRequest("No such fungible token.").Result()
	}

	if !k.IsAuthorised(ctx, owner) {
		return sdkTypes.ErrUnauthorized("Not authorised to freeze token account.").Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	fungibleAccount := k.getFungibleAccount(ctx, symbol, tokenAccount)
	if fungibleAccount == nil {
		return sdkTypes.ErrUnknownRequest("No such token account to freeze.").Result()
	}

	if fungibleAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("Fungible token account already frozen.").Result()
	}

	fungibleAccount.Frozen = true
	fungibleAccount.Metadata = metadata

	k.storeFungibleAccount(ctx, symbol, fungibleAccount)

	eventParam := []string{symbol, tokenAccount.String()}
	eventSignature := "FrozenFungibleTokenAccount(string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token account which must not be freeze.
* Signer must be KYC authorised.
* Action of Re-freeze-item is not allowed.


Last, you define the actual logic for handling the MsgTypeSetFungibleTokenAccountStatus-UnfreezeTokenAccount message in `handleMsgSetFungibleTokenAccountStatus`:

```
func (k *Keeper) UnfreezeFungibleTokenAccount(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, tokenAccount sdkTypes.AccAddress, metadata string) sdkTypes.Result {
	if !k.IsAuthorised(ctx, owner) {
		return sdkTypes.ErrUnauthorized("Not authorised to unfreeze token account.").Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return sdkTypes.ErrUnknownRequest("No such fungible token.").Result()
	}

	fungibleAccount := k.getFungibleAccount(ctx, symbol, tokenAccount)
	if fungibleAccount == nil {
		return sdkTypes.ErrUnknownRequest("No such fungible token account to unfreeze.").Result()
	}

	if !fungibleAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("Fungible token account not frozen.").Result()
	}

	fungibleAccount.Frozen = false
	fungibleAccount.Metadata = metadata

	k.storeFungibleAccount(ctx, symbol, fungibleAccount)

	eventParam := []string{symbol, tokenAccount.String()}
	eventSignature := "UnfreezeFungibleTokenAccount(string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token account which must be freeze.
* Signer must be KYC authorised.
* Action of Re-unfreeze-item is not allowed.



`Events`

1.This tutorial describes how to create maxonrow events for scanner base on freeze token account
after emitted by a network.

```
eventParam := []string{symbol, tokenAccount.String()}
eventSignature := "FrozenFungibleTokenAccount(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using FrozenFungibleTokenAccount(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |



2.This tutorial describes how to create maxonrow events for scanner base on unfreeze token account after emitted by a network.

```
eventParam := []string{symbol, tokenAccount.String()}
eventSignature := "UnfreezeFungibleTokenAccount(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UnfreezeFungibleTokenAccount(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |

