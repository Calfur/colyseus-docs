- [Debug messages](#debug-messages)
- [Inspector (`--inspect` flag)](#inspector)

## Debug messages

To enable all debug logs, run your server using the `DEBUG=colyseus:*` environment variable:

``` bash
DEBUG=colyseus:* npm start
```

| Name | Description |
|------|-------------|
| `colyseus:errors` | Whenever unexpected (or expected, internally) errors happens on the server-side. |
| `colyseus:matchmaking` | Whenever a room is spanwed or disposed. |
| `colyseus:message` | Incoming and outgoing room messages |
| `colyseus:patch` | The number of bytes and interval between patches broadcasted to all clients. |
| `colyseus:connection` | Incoming and outgoing connections |

## Inspector

You can use the the built-in inspector from Node.js to debug your application.

!!! Tip
    Read more about [Debugging Node.js Applications](https://nodejs.org/en/docs/inspector/).

### Using the inspector on production environment

Be careful when using the inspector on production. Using memory snapshots and breakpoints will impact the experience of your users directly.

*1.* Connect to the remote server:

``` bash
ssh root@remote.example.com
```

*2.* Check the PID of the Node process

``` bash
ps aux | grep node
```

*3.* Attach the inspector on the process

``` bash
kill -usr1 PID
```

*4.* Create a SSH tunnel from your local machine to the remote inspector

``` bash
ssh -L 9229:localhost:9229 root@remote.example.com
```

Your production server should now appear on [`chrome://inspect`](`chrome://inspect`).

