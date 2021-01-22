This query the item data by given of symbol and item ID.

After the router is defined, define the inputs and responses for this queryItemData:

```
func queryItemData(cdc *codec.Codec, ctx sdkTypes.Context, path []string, _ abci.RequestQuery, keeper *Keeper) ([]byte, sdkTypes.Error) {
	if len(path) != 2 {
		return nil, sdkTypes.ErrUnknownRequest(fmt.Sprintf("Invalid path %s", strings.Join(path, "/")))
	}

	symbol := path[0]
	itemID := path[1]

	item := keeper.GetNonFungibleItem(ctx, symbol, itemID)
	owner := keeper.GetNonFungibleItemOwnerInfo(ctx, symbol, itemID)

	var itemInfo ItemInfo
	itemInfo.Item = item
	itemInfo.Owner = owner

	js := cdc.MustMarshalJSON(itemInfo)

	return js, nil
}

```

Notes on the above code:

This query request TWO path-parameters which refer to token-symbol and item-id. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of ItemData, the normal ItemData struct is already JSON marshalable, but we need to add a .String() method on it.

`Parameters`

| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


`Example`

In this example, we will explain how to query item data with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/nonFungible/item_data/TNFT/334455",
    	"",
    	"0",
    	false
    	],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this: 
```
{
    "jsonrpc": "2.0",
    "id": 0,
    "result": {
        "response": {
            "code": 0,
            "log": "",
            "info": "",
            "index": "0",
            "key": null,
            "value": "eyJPd25lciI6Im14dzF0Z2ZjZXJ6N2N2eDZzOG5zbHUzNW1nbG16eXFwc2RtcW5mZjI4MCIsIkl0ZW0iOnsiSUQiOiIzMzQ0NTUiLCJQcm9wZXJ0aWVzIjoicHJvcGVydGllcyIsIk1ldGFkYXRhIjoibWV0YWRhdGEiLCJUcmFuc2ZlckxpbWl0IjoiMCIsIkZyb3plbiI6ZmFsc2V9fQ==",
            "proof": null,
            "height": "1242",
            "codespace": ""
        }
    }
}
```
