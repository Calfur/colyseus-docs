# [State Handling](/state/) » Fossil Delta

## FossilDelta is going to be deprecated on future versions of Colyseus. Make sure to use the [Schema Serializer](/state/schema).

The `FossilDeltaSerializer` is permissive about which structure you provide as the state. You may provide a raw object, or a class instance. All the [enumerable properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#Description) of the object provided are going to be serialized to the clients.

- Non-synchronizable properties should be non-enumerable (through [`@nosync`](#non-synchronizable-properties-nosync))
- You may white-list synchronizable propertires through [`toJSON()`](#white-listing-synchronizable-properties-thorugh-tojson) method.

## Server-side

<!-- !!! Tip
    - For synchronization with the client-side, see [client-side state synchronization](/client/state/fossil-delta).
    - See Colyseus' technical approach on [state synchronization](/state/fossil-delta). -->

### Raw Object State

The simplest way to deal with the room state is using a raw JavaScript objects directly in the `Room` handler.

**On the following example, you'll see:**

- Creating player upon client connection.
- Handling client-side actions and updating the state to move `x` position.
- Removing player upon client disconnection.

```typescript fct_label="TypeScript"
import { Room, Client } from "colyseus";

export class BattleRoom extends Room {

  onCreate (options: any) {
    this.setState({
      players: {}
    });

    this.onMessage("left", (client, message) => {
      this.state.players[ client.sessionId ].x -= 1;
    });

    this.onMessage("right", (client, message) => {
      this.state.players[ client.sessionId ].x += 1;
    });
  }

  onJoin (client) {
    this.state.players[ client.sessionId ] = {
      x: 0,
      y: 0
    };
  }

  onLeave (client) {
    delete this.state.players[ client.sessionId ];
  }

}
```

```typescript fct_label="JavaScript"
const colyseus = require('colyseus')

export class BattleRoom extends colyseus.Room {

  onCreate (options: any) {
    this.setState({
      players: {}
    });

    this.onMessage("left", (client, message) => {
      this.state.players[ client.sessionId ].x -= 1;
    });

    this.onMessage("right", (client, message) => {
      this.state.players[ client.sessionId ].x += 1;
    });

  }

  onJoin (client) {
    this.state.players[ client.sessionId ] = {
      x: 0,
      y: 0
    };
  }

  onLeave (client) {
    delete this.state.players[ client.sessionId ];
  }

}
```

### Your Own Data Structures

Whilst it's possible to use raw data directly on [`this.setState()`](/server/room/#setstate-object). The recommended way to handle your state is through your own data structures.

**On the following (rewritten) example, you'll see:**

- A clean `BattleRoom` implementation, directly coupled to the state structure.
- A large data structure holding the entire room state (`BattleState`)
    - Methods manipulating smaller data structures
- A small decoupled data structure representing a single entity (`Player`)

```typescript fct_label="TypeScript"
import { Room, Client } from "colyseus";

class Player {
  x: number;
  y: number;
}

class BattleState {
  players: {[id: string]: Player} = {};

  addPlayer (client) {
    this.players[ client.sessionId ] = new Player(0, 0);
  }

  removePlayer (client) {
    delete this.players[ client.sessionId ];
  }

  movePlayer (client, action) {
    if (action === "left") {
      this.players[ client.sessionId ].x -= 1;

    } else if (action === "right") {
      this.players[ client.sessionId ].x += 1;
    }
  }
}

export class BattleRoom extends Room<BattleState> {

  onCreate (options: any) {
    this.setState(new BattleState());

    this.onMessage("action", (client, message) => {
      this.state.movePlayer(client, message);
    })
  }

  onJoin (client: Client) {
    this.state.addPlayer(client);
  }

  onLeave (client: Client) {
    this.state.removePlayer(client);
  }
}
```


```typescript fct_label="JavaScript"
const colyseus = require('colyseus');

class Player {
  constructor (x, y) {
    this.x = x;
    this.y = y;
  }
}

class BattleState {
  constructor () {
    this.players = {};
  }

  addPlayer (client) {
    this.players[ client.sessionId ] = new Player(0, 0);
  }

  removePlayer (client) {
    delete this.players[ client.sessionId ];
  }

  movePlayer (client, action) {
    if (action === "left") {
      this.players[ client.sessionId ].x -= 1;

    } else if (action === "right") {
      this.players[ client.sessionId ].x += 1;
    }
  }
}

module.exports = class BattleRoom extends colyseus.Room {

  onCreate (options: any) {
    this.setState(new BattleState());

    this.onMessage("action", (client, data) => {
      this.state.movePlayer(client, data.action);
    });
  }

  onJoin (client) {
    this.state.addPlayer(client);
  }

  onLeave (client) {
    this.state.removePlayer(client);
  }

}
```

#### Non-synchronizable properties (`@nosync`)

To prevent private properties from leaking into your clients' state, you need to set those properties as **non-enumerable**. The decorator `@nosync` is a syntax sugar for this purpose.

```typescript fct_label="TypeScript"
import { nosync } from "colyseus";

export class Player {
  x: number = 0;
  y: number = 0;

  @nosync
  wontBeSynched: string = "This property won't be synched with clients";
}
```

```typescript fct_label="JavaScript"
const colyseus = require('colyseus');

module.exports = class Player {
  constructor () {
    this.x = 0;
    this.y = 0;
    this.wontBeSynched = "This property won't be synched with clients";
  }
}

colyseus.nosync(Player.prototype, "wontBeSynched");
```

#### White-listing synchronizable properties thorugh `toJSON()`

In your data structures, you may implement a `toJSON()` method, to explicitly define how it should be serialized for the clients. This way, you don't need to use [`nosync`](#non-synchronizable-properties-nosync), since you're white-listing the properties manually.

```typescript fct_label="TypeScript"
export class Player {
  x: number = 0;
  y: number = 0;
  complexObject = new SomethingElse();

  toJSON () {
    return {
      x: this.x,
      y: this.y,
    }
  }
}
```

```typescript fct_label="JavaScript"
module.exports = class Player {
  constructor (x, y) {
    this.x = x;
    this.y = y;
    this.complexObject = new SomethingElse();
  }

  toJSON () {
    return {
      x: this.x,
      y: this.y,
    }
  }
}
```

### Limitations

#### Avoid using `Map`, `Set`

Avoid using `Map` and `Set` for public, synchronizeable, properties.

Unfortunately, the JavaScript built-in types `Map` and `Set` aren't serializeable. That's because their internal structure is not exposed as enumerable properties:

```typescript
var myMap = new Map([["k1", "v1"], ["k2", "v2"]]);
// => Map(2) {"key" => "value", "key2" => "value2"}
JSON.stringify(myMap);
// => "{}"
```

You're encouraged to use these structures as private, non-synchronizable variables, though. See [`@nosync`](#non-synchronizable-properties-nosync).

#### Avoid mutating arrays

- `push`ing new entries is OK - the clients will receive a single `"add"` operation.
- `pop`ing the last entry is OK - the clients will receive a single `"remove"` operation.

Removing or inserting entries in-between will generate one `"replace"` operation for each entry that had the index changed. Be careful to handle these changes in the client-side properly.

## Client-side

### Listening for changes in a particular variable.

The example below describes how to listen for a particular variable. You may use the full path of a known variable, such as `"a/really/deep/path"`, `"world/fog"`, etc.

```javascript fct_label="JavaScript"
room.listen("currentTurn", (change) => {
    console.log("current turn is now", change.value);
})
```

```csharp fct_label="C#"
using Colyseus;
// ...

room.Listen("currentTurn", OnCurrentTurnChange);

void OnCurrentTurnChange (DataChange change)
{
  Debug.Log ("current turn is now:");
  Debug.Log (change.value);
});
```

```lua fct_label="lua"
room:listen("currentTurn", function(change)
  print("current turn is now: " .. change.value);
end)
```

```haxe fct_label="Haxe"
room.listen("currentTurn", function (change) {
  trace("current turn is now: " + change.value);
});
```

### Listening to map data structures

The example below describes how to catch addition and removals of a map. The callback will be called multiple times, for each entry added or removed from a map of players.

!!! Tip "This example do not listen for changes **inside** each player instance."
    To listen for changes **inside** each player instance, you should listen for [`"players/:id/:attribute"` path as well](#listening-to-attribute-changes-of-deep-data-structures).

```typescript fct_label="JavaScript"
room.listen("players/:id", (change) => {
  if (change.operation === "add") {
    console.log("new player added to the state");
    console.log("player id:", change.path.id);
    console.log("player data:", change.value);

  } else if (change.operation === "remove") {
    console.log("player has been removed from the state");
    console.log("player id:", change.path.id);
  }
});
```

```csharp fct_label="C#"
using Colyseus;
// ...

room.Listen("players/:id", OnPlayerChange);

void OnPlayerChange (DataChange change)
{
  if (change.operation == "add") {
    Debug.Log ("new player added to the state");
    Debug.Log (change.path["id"]);
    Debug.Log (change.value);

  } else if (change.operation == "remove") {
    Debug.Log ("player has been removed from the state");
    Debug.Log (change.path["id"]);
  }
});
```

```lua fct_label="lua"
room:listen("players/:id", function(change)
  if (change.operation == "add") then
    print("new player added to the state")
    print(change.path["id"])
    print(change.value)

  elseif (change.operation == "remove") then
    print("player has been removed from the state")
    print(change.path["id"])
  end
end)
```

```haxe fct_label="Haxe"
room.listen("players/:id", function (change) {
  if (change.operation === "add") {
    trace("new player added to the state");
    trace("player id:" + change.path.id);
    trace("player data:" + Std.string(change.value));

  } else if (change.operation === "remove") {
    trace("player has been removed from the state");
    trace("player id:" + change.path.id);
  }
});
```

### Listening to attribute changes of deep data structures

The example below describes how to catch each attribute change from inside a map of players. The callback will be called multiple times, for each attribute that has changed.

```typescript fct_label="JavaScript"
room.listen("players/:id/:attribute", (change) => {
  console.log(change.operation); // => "add" | "remove" | "replace"
  console.log(change.path.attribute, "has been changed");
  console.log(change.path.id);
  console.log(change.value);
});
```

```csharp fct_label="C#"
using Colyseus;
// ...

room.Listen("players/:id/:attribute", OnPlayerAttributeChange);

void OnPlayerAttributeChange (DataChange change)
{
  Debug.Log (change.operation); // => "add" | "remove" | "replace"
  Debug.Log (change.path["attribute"] + "has been changed");
  Debug.Log (change.path["id"]);
  Debug.Log (change.value);
});
```

```lua fct_label="lua"
room:listen("players/:id/:attribute", function()
  print(change.operation) // => "add" | "remove" | "replace"
  print(change.path["attribute"] + "has been changed")
  print(change.path["id"])
  print(change.value)
end)
```

```haxe fct_label="Haxe"
room.listen("players/:id/:attribute", function(change) {
  trace(change.operation); // => "add" | "remove" | "replace"
  trace(change.path.attribute + " has been changed");
  trace(change.path.id);
  trace(Std.string(change.value));
});
```

### Initial state / Listening to incoming AND existing data in the state

The callbacks will be triggered for each incoming **change** in the state after the moment of registration. To listen also for existing data on the state at the moment you register the callback, make sure to pass `true` on the `immediate` argument.

```typescript fct_label="JavaScript"
room.listen("players/:id", (change) => {
  // ...
}, true); // immediate
```

```csharp fct_label="C#"
using Colyseus;
// ...

room.Listen("players/:id", OnPlayerChange, true); // immediate

void OnPlayerChange (DataChange change)
{
  // ...
});
```

```lua fct_label="lua"
room:listen("players/:id", function(change)
  -- ...
end, true) -- immediate
```

```haxe fct_label="Haxe"
room.listen("players/:id", function (change) {
  // ...
}, true); // immediate
```

<!-- ### Using wildcards

You may use a wildcard (a path segment starting with `:`) in the `listen` method for attributes of unknown name. The matched name will be available at `change.path`.

**Examples**

- Wildcard for a map of entities: (`"entities/:id/:attribute"`)
    - The `:id` of the entity will be available at `change.path.id`
    - The `:attribute` (any attribute change, such as `x`, `y`, etc) of the entity will be available at `change.path.attribute` -->