# psig

psig is a golang package bridge [YoMo](https://github.com/yomorun/yomo) and [Presencejs](https://presence.js.org).
Developers can interact with Presencejs in YoMo serverless functions.

## Usage

1. Generate the YoMo Stateful Serverless:

```sh
yomo init presence-backend-sfn
```

2. Add the `psig` golang package

```sh
go get github.com/yomorun/psig
```

3. Import `psig` in your YoMo Stateful Serverless function:

```go
func Handler(ctx serverless.Context) {
	// Create presencejs context
	pCtx, err := psig.NewContext(ctx, sfnName)
	if err != nil {
		fmt.Println("psig.NewContext:", err)
		return
	}

	// Handle presencejs events
	sig := pCtx.ReadSignalling()

	// sig.Type describe the type of the event, it can be: `Data` or `Control`
	if sig.Type != psig.SigData {
		return
	}

	// the channel of Presencejs
	fmt.Println("presencejs: channel: %v, data, %v", sig.Channel, sig.Payload)


	// Write event to Presencejs, this message will be broadcast to all clients in the channel
	resp, _ := json.Marshal(myResponseObject)

	pCtx.WriteEvent(&psig.ChannelEvent{
		Event: EVENT_COMPLETED,
		Data:  string(resp),
	})
}
```

## API

### NewContext

Returns context for prscd from [serverless.Context](https://yomo.run/docs/api/sfn#sfnsethandlerfn-asynchandler-error). The Context can be used to load events and write events.

### PrscdContext

## License

[Apache 2.0](./LICENSE]
