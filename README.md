# Z-Wave JS Server

Small server wrapper around Z-Wave JS to access it via a WebSocket.

## Trying it out

These instructions are for development only. These CLIs will be available as `zwave-server` and `zwave-client` after installing the NPM package.

### Start server

```shell
ts-node src/bin/server.ts /dev/tty0
```

Opens server on `ws://localhost:3000`.

You can specify a configuration file with `--config`. This can be a JSON file or a JS file that exports the config. It needs to follow the [Z-Wave JS config format](https://zwave-js.github.io/node-zwave-js/#/api/driver?id=zwaveoptions).

If you don't have a USB stick, you can add `--mock` to use a fake stick.

### Start client

Requires server to be running.

Default connects to `ws://localhost:3000`:

```shell
ts-node src/bin/client.ts
```

To specify different host:

```shell
ts-node src/bin/client.ts ws://192.168.1.100:6000
```

To specify that it outputs each message on a single line so it can be replayed later:

```shell
ts-node src/bin/client.ts --dump
```

You can filter the output by a specific node ID:

```shell
ts-node src/bin/client.ts --node 52
```

All these options can be combined.

## API

When a client connects, the server will send the version.

```ts
interface {
  type: "version";
  driverVersion: string;
  serverVersion: string;
  homeId: number;
}
```

To start receive the state and get events, the client needs to send the `start_listening` command.

```ts
interface {
  messageId: string;
  command: "start_listening";
}
```

The server will respond with the current state and start sending events.

```ts
interface {
  type: "result";
  messageId: string; // maps the `start_listening` command
  success: true,
  result: {
    state: {
      controller: Partial<ZWaveController>;
      nodes: Partial<ZWaveNode>[];
    }
  };
}
```

After that, the client will be notified of each state change that happens inside Z-Wave JS.

Event keys follow the names/types as used by Z-Wave JS.

```ts
interface {
  type: "event",
  event: {
    source: "driver" | "controller" | "node";
    event: string;
    [key: string]: unknown;
  }
}
```

## Client commands

```ts
interface {
  messageId: string;
  command: "start_listening";
}
```

```ts
interface {
  messageId: string;
  command: "node.set_value";
  valueId: {
    commandClass: CommandClasses;
    endpoint?: number;
    property: string | number;
    propertyKey?: string | number;
  },
  value: any
}
```

## Authentication

Z-Wave JS Server does not handle authentication and allows all connections to the websocket API. If you want to add authentication, add authentication middleware to your Express instance or run NGINX in front of Express instance.
