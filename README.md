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

Returns `PrscdContext` for prscd from [serverless.Context](https://yomo.run/docs/api/sfn#sfnsethandlerfn-asynchandler-error). This Context can be used to load events and write events.

### PrscdContext

`PrscdContext` is the context for Presencejs. Developers can read events from yomo context and write events to it.

Methods:

- `ReadEvent() (*ChannelEvent, error)`: Read event from Presencejs.
- `WriteEvent(event *ChannelEvent) error`: Write event to Presencejs.
- `ReadSignalling() *Signalling`: Read signalling from Presencejs.

### Signalling

`Signalling` is the data structure of Presencejs.

```go
type Signalling struct {
	Type    string `msgpack:"t"`              // Type describes the type of signalling, `Data Signal` or `Control Signal`
	OpCode  string `msgpack:"op,omitempty"`   // OpCode describes the operation type of signalling
	Channel string `msgpack:"c"`              // Channel describes the channel
	Sid     string `msgpack:"sid,omitempty"`  // Sid describes the peer id on this node in backend
	Payload []byte `msgpack:"pl,omitempty"`   // Payload describes the payload data of signalling
	Cid     string `msgpack:"p"`              // Cid describes the client id of peer, set by developer
	AppID   string `msgpack:"app,omitempty"`  // AppID describes the app_id
	MeshID  string `msgpack:"mesh,omitempty"` // MeshID describes the mesh_id of this node
}
```

### ChannelEvent

`ChannelEvent` describes the event data structure of Presencejs. It contains the event type and data. It's used in `channel.broadcast()` and `channel.subscribe()` in Presencejs.

Properties:

```go
type ChannelEvent struct {
	Event string `msgpack:"event"`
	Data  string `msgpack:"data"`
}
```

### Constants

- `OpChannelJoin`: when a peer joins the channel, client will sends this event to prscd server, ask for joining the channel.
- `OpPeerOffline`: when a peer leaves the channel, this event will be sent to all peers in the channel.
- `OpPeerOnline`: when a peer joins the channel, this event will be sent to all peers in the channel.
- `OpState`: when a peer syncs state changes, this event will be sent to all peers in the channel.

## License

[Apache 2.0](./LICENSE]
