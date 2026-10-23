简单的桌面设置：

```js
const { app, BrowserWindow } = require("electron");

let mainWindow;

function createWindow() {
    mainWindow = new BrowserWindow({
        width: 1200,
        height: 800,
        webPreferences: {
            nodeIntegration: true,
        },
    });

    mainWindow.loadURL("http://localhost:3000");

    mainWindow.on("closed", () => {
        mainWindow = null;
    });
}

app.whenReady().then(createWindow);

app.on("window-all-closed", () => {
    if (process.platform !== "darwin") {
        app.quit();
    }
});

app.on("activate", () => {
    if (mainWindow === null) {
        createWindow();
    }
});

```


package.json
```json
{
  "name": "frontend",
  "version": "0.0.1",
  "main": "main.js",
  "homepage": ".",
  "private": true,
  "dependencies": {
    "@tinymce/tinymce-react": "^4.3.2",
    "antd": "^5.13.2",
    "axios": "^1.6.5",
    "concurrently": "^8.2.2",
    "cross-env": "^7.0.3",
    "electron": "^28.1.4",
    "electron-is-dev": "^3.0.1",
    "react": "^18.2.0",
    "react-beautiful-dnd": "^13.1.1",
    "react-dom": "^18.2.0",
    "react-modal": "^3.16.1",
    "react-router-dom": "^6.21.3",
    "react-scripts": "5.0.1",
    "uuid": "^9.0.1",
    "wait-on": "^7.2.0"
  },
  "scripts": {
    "start": " react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "dev": "concurrently \" wait-on http://localhost:3000 && electron . \" \" cross-env BROWSER=none npm start \" "
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}

```


