This is the message type used to update the status of a non-fungible token, eg. Approve, Reject, Freeze or unfreeze, Approve-transfer-ownership, Reject-transfer-ownership. This transaction require `issuer`, `provider` and `middleware` signature.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Item owner| | 
| payload | ItemPayload | true   | Item Payload information| | 
| signatures | []Signature | true   | Array of Signature| | 


Item Payload Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| token | TokenData | true   | Token data| | 
| pub_key | nil | true   | crypto.PubKey| | 
| signature | []byte | true   | signature| | 


Token Data Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Token owner| | 
| nonce | string | true   | nonce signature| | 
| status | string | true   | There are different type of status, which include `APPROVE`, `REJECT`, `FREEZE`, `UNFREEZE`, `APPROVE_TRANFER_TOKEN_OWNERSHIP`, `REJECT_TRANFER_TOKEN_OWNERSHIP`. All this keywords must be matched while come to this message type | | 
| symbol | string | true   | Token-symbol| | 
| transferLimit | string | true   | Item transfer Limit setting| | 
| mintLimit | string | true   | Item mint Limit setting| | 
| burnable | bool | true   | flag of Item burnable setting, either `TRUE` or `FALSE`| | 
| transferable | bool | true   | flag of Item transferable setting, either `TRUE` or `FALSE`| | 
| modifiable | bool | true   | flag of Item modifiable setting, either `TRUE` or `FALSE`| |
| pub | bool | true   | flag of public setting, either `TRUE` or `FALSE`| | 
| tokenFees | []TokenFee | true   | Fee Setting information| | 
| endorserList | []string | true   | Endorser list| | 


`status`

| status | Details                 |
| -------- | --------------------------- |
| APPROVE  | If status set to `APPROVE`, this is to approve the token. Once approved, only then the token able to perform token action for example: mint, burn, transfer and etc. Additionally, the properties, transferLimit, mintLimit, burnable, transferable, modifiable, pub is mutable.| |
| REJECT  | If status set to `REJECT`, this is to reject the token. Once rejected, the token NOT allowed to perform any token action like: mint, burn, transfer and etc.| | 
| FREEZE  | If status set to `FREEZE`, this is to freeze the token. Once frozen, the token NOT allowed to perform any token action like: mint, burn and so on. | | 
| UNFREEZE  | If status set to `UNFREEZE`, this is to unfreeze the token. Once unfreeze, only then the token allowed to perform token action like mint, burn, transfer and so on.| |
| APPROVE_TRANFER_TOKEN_OWNERSHIP  | If status set to `APPROVE_TRANFER_TOKEN_OWNERSHIP`, this is to approve the transfer-ownership of the token. Once approved, only then the new owner can accept the ownership of token.| | 
| REJECT_TRANFER_TOKEN_OWNERSHIP  | If status set to `REJECT_TRANFER_TOKEN_OWNERSHIP`, this is to reject the transfer-ownership of the token. Once rejected, can not do transfer-token-ownership to new party.| |   

`transferLimit`

* User must input the number as string type during the process of set APPROVE of a token-symbol. 
* Once exceeded this setting limits, will received an alert of `Transfer limit existed`. 
* If transfer-limit is set to ZERO, that means not allowed to do any transfer. 
* Upon the process of approval, User NOT allowed to update this setting again.

`mintLimit`

* User must input the number as string type during the process of set APPROVE of a token-symbol. 
* Once exceeded this setting limits, will received an alert of `Mint limit existed`. 
* If mint-limit is set to ZERO, that means is NO LIMITATION on the minting for this.
* Upon the process of approval, User NOT allowed to update this setting again.


`burnable`

* For the burnable equals `TRUE`, user allowed to proceed during burn-item process. 
* For the burnable equals `FALSE`, user NOT allowed to proceed during burn-item process with an alert of `Invalid token action`. 
* Upon the process of approval, User NOT allowed to update this setting again.

`transferable`

* For the transferable equals `TRUE`, user allowed to proceed during transfer-item process.
* For the transferable equals `FALSE`, user NOT allowed to proceed during transfer-item process with an alert of `Invalid token action`.
* Upon the process of approval, User NOT allowed to update this setting again.


`modifiable`

* For the modifiable equals `TRUE`, will validate Item Owner must equals to relevant signer during Update Item Metadata process. If not, with an alert of `Token item not modifiable`.
* For the modifiable equals `FALSE`, will validate signer must equals to token-Owner during Update Item Metadata process. If not, with an alert of `Token item not modifiable`. 
* Upon the process of approval, User NOT allowed to update this setting again.


`pub`

* For the public equals `TRUE`, will validate minter must equals to `new item-owner` during Mint Item process.
If not, with an alert of `Public token can only be minted to oneself`.
* For the public equals `FALSE`, will validate minter must equals to `token-owner` during Mint Item process. 
If not, with an alert of `Invalid token minter`.
* Upon the process of approval, User NOT allowed to update this setting again.



`tokenFees`

* This input value is compulsory while come to process of approve-token. 
* The feeName value will be set for different action types which inside the tokenPayload of the current token : eg. transfer-item, mint-item, burn-item, transfer-token-Ownership, accept-token-Ownership. 
* Example of this can refer to `https://alloys-rpc.maxonrow.com/debug/fee_info?`
* Upon the process of approval, User NOT allowed to update this setting again.


`endorserList`

* User can provide the list with the correct wallet address. Invalid signer that NOT in endorser list will received an alert of `Invalid endorser`. 
* If endorser list is set to empty, then anyone with valid wallet address can proceed for the endorsement-item process. 


Token Fee Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| action | string | true   | action | | 
| feeName | string | true   | fee setting| | 



Example
```
{
    "type": "nonFungible/setNonFungibleTokenStatus",
    "value": {
        "owner": "mxw1md4u2zxz2ne5vsf9t4uun7q2k0nc3ly5g22dne",
        "payload": {
            "token": {
                "from": "mxw1f8r0k5p7s85kv7jatwvmpartyy2j0s20y0p0yk",
                "nonce": "0",
                "status": "APPROVE",
                "symbol": "TNFT",
                "transferLimit": "2",
                "mintLimit": "2",
                "burnable": true,
                "transferable": true,
                "modifiable": true,
                "pub": true,
                "tokenFees": [
                    {
                        "action": "transfer",
                        "feeName": "default"
                    },
                    {
                        "action": "mint",
                        "feeName": "default"
                    },
                    {
                        "action": "burn",
                        "feeName": "default"
                    },
                    {
                        "action": "transferOwnership",
                        "feeName": "default"
                    },
                    {
                        "action": "acceptOwnership",
                        "feeName": "default"
                    }
                ],
                "endorserList": [
                    "mxw1f8r0k5p7s85kv7jatwvmpartyy2j0s20y0p0yk",
                    "mxw1k9sxz0h3yeh0uzmxet2rmsj7xe5zg54eq7vhla"
                ]
            },
            "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A0VBHXKgUEU2fttqh8Lhqp1G6+GzOxTXvCExzDLEdfD7"
            },
            "signature": "Hiaih1n5Y1/gJQQDKbXmskPPEXzP2MOu+mxRDnJEBXxFtCQeRe3tK5cN7QTl326YpUMvsejG9Go+u4rzNaqILg=="
        },
        "signatures": [
            {
                "pub_key": {
                    "type": "tendermint/PubKeySecp256k1",
                    "value": "Ausyj7Gas2WkCjUpM8UasCcezXrzTMTRbPHqYx44GzLm"
                },
                "signature": "tIRCsR4cy+Kp1IOCOmoBqM8xr/nnnsNGF2V/6QX+UORtx/HNaQ9+HtYkaYGVJ5I4T5yMHXKwtgDU8IpyHD6l/w=="
            }
        ]
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgSetNonFungibleTokenStatus` message is received.

In the file (./x/token/nonfungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateNonFungibleToken:
			return handleMsgCreateNonFungibleToken(ctx, keeper, msg)
		'case MsgSetNonFungibleTokenStatus:
			return handleMsgSetNonFungibleTokenStatus(ctx, keeper, msg)'
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

First, you define the actual logic for handling the `MsgSetNonFungibleTokenStatus` - ApproveToken message in `handleMsgSetNonFungibleTokenStatus`:

```
func (k *Keeper) approveNonFungibleToken(ctx sdkTypes.Context, symbol string, tokenFees []TokenFee, mintLimit, transferLimit sdkTypes.Uint, signer sdkTypes.AccAddress, endorserList []sdkTypes.AccAddress, burnable, transferable, modifiable, public bool) sdkTypes.Result {
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

	token.Flags.AddFlag(ApprovedFlag)
	if burnable {
		token.Flags.AddFlag(BurnFlag)
	}
	if transferable {
		token.Flags.AddFlag(TransferableFlag)
	}
	if modifiable {
		token.Flags.AddFlag(ModifiableFlag)
	}
	if public {
		token.Flags.AddFlag(PubFlag)
	}

	token.TransferLimit = transferLimit
	token.MintLimit = mintLimit
	token.EndorserList = endorserList

	k.storeToken(ctx, symbol, token)

	// Event: Approved fungible token
	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "ApprovedNonFungibleToken(string,string)"
	events := types.MakeMxwEvents(eventSignature, signer.String(), eventParam)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: events,
		Log:    resultLog.String(),
	}

}

```

In this function, requirements need to be met before emitted by the network.  

* A valid Token.
* Token must not be approved.
* Signer must be KYC authorised.
* Action of Re-approved is not allowed.


Second, you define the actual logic for handling the MsgSetNonFungibleTokenStatus-RejectToken message in handleMsgSetNonFungibleTokenStatus:

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
	store.Delete(tokenTypeKey)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "RejectedNonFungibleToken(string,string)"

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


Thirth, you define the actual logic for handling the MsgSetNonFungibleTokenStatus-FreezeToken message in handleMsgSetNonFungibleTokenStatus:

```
func (k *Keeper) freezeNonFungibleToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress) sdkTypes.Result {
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
	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "FrozenNonFungibleToken(string,string)"

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


Next, you define the actual logic for handling the MsgSetNonFungibleTokenStatus-UnfreezeToken message in handleMsgSetNonFungibleTokenStatus:

```
func (k *Keeper) unfreezeNonFungibleToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress) sdkTypes.Result {

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

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "UnfreezeNonFungibleToken(string,string)"

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

Next, you define the actual logic for handling the MsgSetNonFungibleTokenStatus-ApproveTransferTokenOwnership message in handleMsgSetNonFungibleTokenStatus:

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
	eventSignature := "ApprovedTransferNonFungibleTokenOwnership(string,string,string)"

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

Last, you define the actual logic for handling the MsgSetNonFungibleTokenStatus-RejectTransferTokenOwnership message in handleMsgSetNonFungibleTokenStatus:

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
	eventSignature := "RejectedTransferNonFungibleTokenOwnership(string,string,string)"

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
eventSignature := "ApprovedNonFungibleToken(string,string)"
events := types.MakeMxwEvents(eventSignature, signer.String(), eventParam)

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: events,
    Log:    resultLog.String(),
}

```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using ApprovedNonFungibleToken(string,string)
* signer : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 


2.This tutorial describes how to create maxonrow events for scanner base on reject token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "RejectedNonFungibleToken(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```
Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using RejectedNonFungibleToken(string,string)
* signer : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 


3.This tutorial describes how to create maxonrow events for scanner base on freeze token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "FrozenNonFungibleToken(string,string)"

accountSequence := signerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using FrozenNonFungibleToken(string,string)
* signer : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 


4.This tutorial describes how to create maxonrow events for scanner base on unfreeze token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "UnfreezeNonFungibleToken(string,string)"

accountSequence := signerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}

```


Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using UnfreezeNonFungibleToken(string,string)
* signer : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 



5.This tutorial describes how to create maxonrow events for scanner base on approve transfer token-ownership after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
eventSignature := "ApprovedTransferNonFungibleTokenOwnership(string,string,string)"

accountSequence := fromWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using ApprovedTransferNonFungibleTokenOwnership(string,string,string)
* from : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 
| newOwner | string | New token owner| | 


6.This tutorial describes how to create maxonrow events for scanner base on reject transfer token-ownership after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
eventSignature := "RejectedTransferNonFungibleTokenOwnership(string,string,string)"

accountSequence := fromWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using RejectedTransferNonFungibleTokenOwnership(string,string,string)
* from : Signer
* eventParam : Event Parameters as below 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| | 
| owner | string | Token owner| | 
| newOwner | string | New token owner| | 
