# Electron-ipc

`Electron-ipc` is a module that makes communication between different processes easier and more convenient, out of the box!

**[中文文档](readme_zh.md)**

## feature

- Two-way communication from main process to rendering process
- Two-way communication from renderer process to main process
- Render process to render process bidirectional communication

## use

### The main process communicates with the rendering process

> main.ts (Main Process)


```typescript
import { app, BrowserWindow, Menu, ipcMain } from "electron";
import path from "path";
import ipc from "savage-electron-ipc";

function createWindow() {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, "preload.ts"),
      // This option needs to be enable, otherwise preload cannot access the node module
      nodeIntegration: true,
    },
  });

  // Add windows that need to communicate, this step is very important
  ipc.addToChannel(mainWindow);

  ipc
    .send<string>("msg", "hello")
    .then((res) => {
      console.log(res);
    })
    .catch((err) => {
      console.log(err);
    });
  mainWindow.loadFile("index.html");
}
// ...
```

> preload.ts (Preload Script)


```typescript
import ipc from "savage-electron-ipc";

ipc.renderFromMain("msg", (e, arg) => {
  console.log(arg);
  return "hi,there!";
});
```

### The rendering process communicates with the main process

> main.ts (Main Process)


```typescript
import { app, BrowserWindow, Menu, ipcMain } from "electron";
import path from "path";
import ipc from "savage-electron-ipc";

function createWindow() {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, "preload.ts"),
    },
  });

  // Add windows that need to communicate, this step is very important
  ipc.addToChannel(mainWindow);

  ipc.receive("msg", (e, v) => {
    console.log(v); // 'hello'
    return "how dare you!";
  });
  mainWindow.loadFile("index.html");
}
// ...
```

> preload.ts (Preload Script)


```typescript
import ipc from "savage-electron-ipc";

ipc.send("msg", "hello");
```

### Rendering process communicates with rendering process

> main.ts (Main Process)


```typescript
import { app, BrowserWindow, Menu, ipcMain } from "electron";
import path from "path";
import ipc from "savage-electron-ipc";

function createWindow() {
  const mainWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, "preload.ts"),
    },
  });

  const secondWindow = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, "preload.ts"),
    },
  });

  // Add windows that need to communicate, this step is very important
  ipc.addToChannel([mainWindow, secondWindow]);

  mainWindow.loadFile("index.html");
  secondWindow.loadFile("index.html");
}
// ...
```

> preload.ts (Preload Script)


```typescript
import ipc from "savage-electron-ipc";

ipc.send("msg", "hello");
```

> preload2.ts (Preload Script)


```typescript
import ipc from "savage-electron-ipc";

ipc.receive("msg", (e, v) => {
  console.log(v); // 'hello'
  return "how dare you!";
});
```



