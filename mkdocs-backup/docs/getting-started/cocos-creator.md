# Cocos Creator 3  (JavaScript SDK)

[Cocos Creator](https://cocos.com/creator) is a Cross-Platform 2D/3D Game Creation Tool.

The Colyseus SDK for Cocos Creator works in the same way as the [JavaScript SDK](/getting-started/javascript-client/). The major difference is how to [import it from a TypeScript Component](#importing-from-the-extension) asset from Cocos Creator.

!!! tip "Looking for an example?"
    Take a look at the [Tic Tac Toe Example](https://github.com/colyseus/cocos-demo-tictactoe).

## 1. Install the extension

- [Open the Cocos Store](https://store.cocos.com/app/en/detail/5086) (Menu: "Extension" &raquo; "Store")
- Search for "Colyseus SDK"
- Download the extension
- Decompress the zip file to <path_to_your_project>/extensions/
![image](https://user-images.githubusercontent.com/42052366/130320623-9b1e556b-4fe1-44a6-b0d7-1e44cc231555.png)
- Start up Cocos Creator, open extention manager (Menu: "Extension" &raquo; "Extension Manager")
- Click "search for plugins" button on the "Project" tab
- Enable the "colyseus-sdk" plugin
![image](https://user-images.githubusercontent.com/42052366/130320703-74e6ea85-e217-462a-ac77-01e933475257.png)


## 2. Update your `tsconfig.json` file

Make sure you have `"esModuleInterop": true` under `"compilerOptions"`:

```json
  // ...

  /* Add your custom configuration here. */
  "compilerOptions": {
    // ...
    "esModuleInterop": true
    // ...
  }
  // ...
```

Reboot your Cocos Creator editor.

## 3. Importing from the extension

You must import Colyseus from its extension path:

```typescript
import Colyseus from 'db://colyseus-sdk/colyseus.js';
```

Below is a more elaborate example you can copy and paste into your project.

Save the contents below as `NetworkManager.ts` under your `assets` folder.

```typescript
import { _decorator, Component, Node } from 'cc';
const { ccclass, property } = _decorator;

import Colyseus from 'db://colyseus-sdk/colyseus.js';

@ccclass('NetworkManager')
export class NetworkManager extends Component {
    @property hostname = "localhost";
    @property port = 2567;
    @property useSSL = false;

    client!: Colyseus.Client;
    room!: Colyseus.Room;

    start () {
        // Instantiate Colyseus Client
        // connects into (ws|wss)://hostname[:port]
        this.client = new Colyseus.Client(`${this.useSSL ? "wss" : "ws"}://${this.hostname}${([443, 80].includes(this.port) || this.useSSL) ? "" : `:${this.port}`}`);

        // Connect into the room
        this.connect();
    }

    async connect() {
        try {
            this.room = await this.client.joinOrCreate("my_room");

            console.log("joined successfully!");
            console.log("user's sessionId:", this.room.sessionId);

            this.room.onStateChange((state) => {
                console.log("onStateChange: ", state);
            });

            this.room.onLeave((code) => {
                console.log("onLeave:", code);
            });

        } catch (e) {
            console.error(e);
        }
    }
}
```

You can now attach `NetworkManager` as a Component on any Node from Cocos Creator.

Because we're using the `@property` decorator above, you can edit the `hostname`, `port` and `useSSL` from Cocos Creator editor:

![Colyseus SDK on TypeScript Component](cocos-creator-component.png)
![login](logged_in.jpg)


---

## Alternative: How to manually add the `colyseus.js` file into your project (without the extension)

- [Download the latest `colyseus-sdk.zip` release from GitHub](https://github.com/colyseus/colyseus.js/releases).
- Unzip the `colyseus-sdk.zip` file into the `assets/` folder.
- Move both `colyseus-cocos-creator.js` and `colyseus.d.ts` files into your Cocos Creator project's `scripts` folder.
- Click on the `colyseus-cocos-creator.js` file from the Assets panel, and enable to "Import As Plugin" (see image below)
- **TypeScript**: require it using `import Colyseus from "./colyseus-cocos-creator.js";`
- **JavaScript**: require it using `const Colyseus = require("./colyseus-cocos-creator.js");`

![Import as plugin](cocos-creator-import-as-plugin.png)
