This is the message type to update the status of a fungible token, eg. Approve, Reject, Freeze or unfreeze, Approve-transfer-ownership, Reject-transfer-ownership. This transaction require `issuer`, `provider` and `middleware` signature.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Token owner| |
| payload | Payload | true   | Payload information| |
| signatures | []Signature | true   | Array of Signature| |


Payload Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| token | TokenData | true   | Token data| |
| pub_key | nil | true   | crypto.PubKey| |
| signature | []byte | true   | Signature| |


Token Data Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Token owner| |
| nonce | string | true   | nonce signature| |
| status | string | true   | There are different type of status, which include `APPROVE`, `REJECT`, `FREEZE`, `UNFREEZE`, `APPROVE_TRANFER_TOKEN_OWNERSHIP`, `REJECT_TRANFER_TOKEN_OWNERSHIP`. All this keywords must be matched while come to this message type | |
| symbol | string | true   | Token-symbol| |
| burnable | bool | true   | flag of token burnable setting, either TRUE or FALSE| |
| tokenFees | []TokenFee | true   | Fee Setting information| | 


`status`

| status | Details                 |
| -------- | --------------------------- |
| APPROVE  | If status set to `APPROVE`, this is to approve the token. Once approved, only then the token able to perform token action for example: mint, burn, transfer and etc. Additionally, the properties, burnable is mutable.| |
| REJECT  | If status set to `REJECT`, this is to reject the token. Once rejected, the token NOT allowed to perform any token action like: mint, burn, transfer and etc.| | 
| FREEZE  | If status set to `FREEZE`, this is to freeze the token. Once frozen, the token NOT allowed to perform any token action like: mint, burn and so on. | | 
| UNFREEZE  | If status set to `UNFREEZE`, this is to unfreeze the token. Once unfreeze, only then the token allowed to perform token action like mint, burn, transfer and so on.| |
| APPROVE_TRANFER_TOKEN_OWNERSHIP  | If status set to `APPROVE_TRANFER_TOKEN_OWNERSHIP`, this is to approve the transfer-ownership of the token. Once approved, only then the new owner can accept the ownership of token.| | 
| REJECT_TRANFER_TOKEN_OWNERSHIP  | If status set to `REJECT_TRANFER_TOKEN_OWNERSHIP`, this is to reject the transfer-ownership of the token. Once rejected, the token NOT allowed to perform any action.| |   


`burnable`

* For the burnable equals `TRUE`, user allowed to proceed during burn process. 
* For the burnable equals `FALSE`, user NOT allowed to proceed during burn process with an alert of `Invalid token action`. 
* Upon the process of approval, User NOT allowed to update this setting again.

`tokenFees`

* This input value is compulsory while come to process of approve-token. 
* The feeName value will be set for different action types which inside the tokenPayload of the current token : eg. transfer, mint, burn, transfer-token-Ownership, accept-token-Ownership. 
* Example of this can refer to `https://alloys-rpc.maxonrow.com/debug/fee_info?`
* Upon the process of approval, User NOT allowed to update this setting again.



Token Fee Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| action | string | true   | Action | |
| feeName | string | true   | Fee setting| |



Example
```
 {
    "type": "token/setFungibleTokenStatus",
    "value": {
        "owner": "mxw1j4duwuaqdj2na054rmlg3pdzncpmwzdjwtfqht",
        "payload": {
            "token": {
                "from": "mxw1lmym5599yja76d2s463390he22pcpng3zzpx4p",
                "nonce": "0",
                "status": "FREEZE",
                "symbol": "TT-4",
                "burnable": true
            },
            "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A8I4AHjOPpkQhEFy24L8SZkwd9UxQyEyudZYcYl9f4jK"
            },
            "signature": "2rX+NvGxvB5vmEa7aeOk3nICZuUcOpv3jgRn64PtGVoT3e9Nq1mBDSUtggr5XyWJFNA6ntBzt/yvL4K+MgW+ng=="
        },
        "signatures": [
            {
                "pub_key": {
                    "type": "tendermint/PubKeySecp256k1",
                    "value": "A4PwoBS8fl/2W+V1HXrWQBn5jXci5gnYUTQzDZFqD0vl"
                },
                "signature": "y92hR1sSHk53B1tSbybOE0q2VQ9yzHnxW0heTk64Q9wHuk5lnQhqoztfM+IqaGOMbUYc3MGVhntkAVUVoXKw8A=="
            }
        ]
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgSetFungibleTokenStatus` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		'case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)'
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

First, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-ApproveToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) approveFungibleToken(ctx sdkTypes.Context, symbol string, tokenFees []TokenFee, burnable bool, metadata string, signer sdkTypes.AccAddress) sdkTypes.Result {
	var token = new(Token)
	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	if token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenAlreadyApproved(symbol).Result()
	}

	// Assign fee to token
	for _, tokenFee := range tokenFees {
		if !k.feeKeeper.FeeSettingExists(ctx, tokenFee.FeeName) {
			return types.ErrFeeSettingNotExists(tokenFee.FeeName).Result()
		}
		err := k.feeKeeper.AssignFeeToTokenAction(ctx, tokenFee.FeeName, token.Symbol, tokenFee.Action)
		if err != nil {
			return err.Result()
		}
	}

	var flags types.Bitmask

	// if fixed supply: fixed supply doesn't allow to mint.
	if !token.Flags.HasFlag(MintFlag) {
		if burnable {
			flags = FixedSupplyBurnableFungibleTokenMask
		} else {
			flags = FixedSupplyNotBurnableFungibleTokenMask
		}
	} else {
		flags = DynamicFungibleTokenMask
	}

	token.Flags = flags + ApprovedFlag
	token.Metadata = metadata

	account := k.getFungibleAccount(ctx, symbol, token.Owner)
	if account == nil {
		account = k.createFungibleAccount(ctx, symbol, token.Owner)
	}

	if account.Frozen {
		return sdkTypes.ErrInternal("Fungible token account is frozen.").Result()
	}

	// if fixed supply: fixed supply doesn't allow to mint.
	if !token.Flags.HasFlag(MintFlag) {
		addFungibleTokenErr := k.addFungibleToken(ctx, symbol, token.Owner, token.MaxSupply)
		if addFungibleTokenErr != nil {
			return addFungibleTokenErr.Result()
		}

		token.TotalSupply = token.TotalSupply.Add(token.MaxSupply)
	}

	k.storeToken(ctx, symbol, token)

	var transferEvents sdkTypes.Events
	if !account.Balance.IsZero() {
		// Event: After approve, added total supply into token owner account.
		transferEventParam := []string{symbol, "mxw000000000000000000000000000000000000000", ownerWalletAccount.GetAddress().String(), token.TotalSupply.String()}
		transferEventSignature := "TransferredFungibleToken(string,string,string,bignumber)"
		transferEvents = types.MakeMxwEvents(transferEventSignature, "mxw000000000000000000000000000000000000000", transferEventParam)
	}
	// Event: Approved fungible token
	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "ApprovedFungibleToken(string,string)"
	events := types.MakeMxwEvents(eventSignature, signer.String(), eventParam)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: events.AppendEvents(transferEvents),
		Log:    resultLog.String(),
	}

}

```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be approved.
* Signer must be KYC authorised.
* Action of Re-approved is not allowed.


Second, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-RejectToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) RejectToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress) sdkTypes.Result {

	var token = new(Token)

	if !k.IsAuthorised(ctx, signer) {
		return sdkTypes.ErrUnauthorized("Not authorised to reject").Result()
	}

	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	var isApproved = token.Flags.HasFlag(ApprovedFlag)
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	if isApproved {
		return types.ErrTokenAlreadyApproved(symbol).Result()
	}

	store := ctx.KVStore(k.key)

	tokenTypeKey := getTokenKey(symbol)
	store.Delete([]byte(tokenTypeKey))

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "RejectedFungibleToken(string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be approved.
* Signer must be KYC authorised.
* Action of Re-reject is not allowed.


Thirth, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-FreezeToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) freezeFungibleToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress, metadata string) sdkTypes.Result {
	var token = new(Token)
	k.mustGetTokenData(ctx, symbol, token)

	signerAccount := k.accountKeeper.GetAccount(ctx, signer)
	if signerAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	token.Flags.AddFlag(FrozenFlag)
	token.Metadata = metadata
	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "FrozenFungibleToken(string,string)"

	accountSequence := signerAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be freeze
* Signer must be KYC authorised.
* Action of Re-freeze is not allowed.

Next, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-UnfreezeToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) unfreezeFungibleToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress, metadata string) sdkTypes.Result {

	var token = new(Token)
	k.mustGetTokenData(ctx, symbol, token)

	signerAccount := k.accountKeeper.GetAccount(ctx, signer)
	if signerAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	if !token.Flags.HasFlag(FrozenFlag) {
		return sdkTypes.ErrUnknownRequest("Fungible token is not frozen.").Result()
	}

	token.Flags.RemoveFlag(FrozenFlag)
	token.Metadata = metadata

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "UnfreezeFungibleToken(string,string)"

	accountSequence := signerAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must be freeze.
* Signer must be KYC authorised.
* Action of Re-unfreeze is not allowed.

Next, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-ApproveTransferTokenOwnership message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) ApproveTransferTokenOwnership(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress) sdkTypes.Result {
	if !k.IsAuthorised(ctx, from) {
		return sdkTypes.ErrUnauthorized("Not authorised to accept transfer token ownership.").Result()
	}

	fromWalletAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	if !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	token.Flags.AddFlag(AcceptTokenOwnershipFlag)
	token.Flags.AddFlag(ApproveTransferTokenOwnershipFlag)

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
	eventSignature := "ApprovedTransferTokenOwnership(string,string,string)"

	accountSequence := fromWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token with TransferTokenOwnership flag equals to true.
* Signer must be KYC authorised.
* Action of Re-approve transfer token-ownership is not allowed.

Last, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-RejectTransferTokenOwnership message in handleMsgSetFungibleTokenStatus:

```
func (k *Keeper) RejectTransferTokenOwnership(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress) sdkTypes.Result {
	if !k.IsAuthorised(ctx, from) {
		return sdkTypes.ErrUnauthorized("Not authorised to accept transfer token ownership.").Result()
	}

	fromWalletAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	if !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	token.Flags.RemoveFlag(TransferTokenOwnershipFlag)

	var emptyAccAddr sdkTypes.AccAddress
	token.NewOwner = emptyAccAddr

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
	eventSignature := "RejectedTransferTokenOwnership(string,string,string)"

	accountSequence := fromWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token TransferTokenOwnership flag equals to true.
* Signer must be KYC authorised.
* Action of Re-reject transfer token-ownership is not allowed.



`Events`

1.This tutorial describes how to create maxonrow events for scanner base on approve token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "ApprovedFungibleToken(string,string)"
events := types.MakeMxwEvents(eventSignature, signer.String(), eventParam)

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: events.AppendEvents(transferEvents),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using ApprovedFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


2.This tutorial describes how to create maxonrow events for scanner base on reject token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "RejectedFungibleToken(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```
Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using RejectedFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


3.This tutorial describes how to create maxonrow events for scanner base on freeze token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "FrozenFungibleToken(string,string)"

accountSequence := signerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using FrozenFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


4.This tutorial describes how to create maxonrow events for scanner base on unfreeze token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "UnfreezeFungibleToken(string,string)"

accountSequence := signerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UnfreezeFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


5.This tutorial describes how to create maxonrow events for scanner base on approve transfer token-ownership after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
eventSignature := "ApprovedTransferTokenOwnership(string,string,string)"

accountSequence := fromWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using ApprovedTransferTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |


6.This tutorial describes how to create maxonrow events for scanner base on reject transfer token-ownership after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
eventSignature := "RejectedTransferTokenOwnership(string,string,string)"

accountSequence := fromWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using RejectedTransferTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |

