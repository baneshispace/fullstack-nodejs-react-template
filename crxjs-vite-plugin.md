# crxjs-vite-plugin

Yes, I'm familiar with the **Vite Plugin for Chrome Extensions** (`crxjs/vite-plugin`), which simplifies building **Chrome extensions** using Vite. It's a great fit for your setup because it allows you to use Vite to bundle and serve your Chrome extension, leveraging its fast development features and modern JavaScript tooling (React, TypeScript, etc.).

Here's how to integrate the **`crxjs/vite-plugin`** into your current setup to streamline Chrome extension development.

### Steps to Set Up Chrome Extension with `crxjs/vite-plugin` and Vite

#### 1. Install `crxjs/vite-plugin`
First, navigate to your **frontend** directory (since you're using Vite for the frontend), and install the `crxjs/vite-plugin` along with any necessary dependencies:

```bash
cd frontend
pnpm add @crxjs/vite-plugin
```

#### 2. Configure Vite for Chrome Extensions
In your **`vite.config.ts`**, integrate the `crxjs/vite-plugin` to build the Chrome extension.

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { crx } from '@crxjs/vite-plugin';
import manifest from './manifest.json';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    crx({ manifest })
  ],
  server: {
    proxy: {
      '/api': 'http://localhost:5000',
    },
  },
});
```

This configuration does the following:
- Uses `@crxjs/vite-plugin` to bundle the Chrome extension.
- The `crx` plugin reads the **manifest** from a `manifest.json` file in the root, which we'll define next.
- `react()` allows for building React components as usual.
- You can also add the proxy configuration to route API calls to your backend (running on port `5000`).

#### 3. Define the Manifest (`manifest.json`)
Create a **`manifest.json`** file in the **frontend** directory for your Chrome extension:

```json
{
  "manifest_version": 3,
  "name": "ChatGPT Template Extension",
  "version": "1.0",
  "description": "Interact with ChatGPT using predefined templates via WebSocket and a local server.",
  "permissions": [
    "storage",
    "activeTab",
    "scripting"
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },
  "host_permissions": [
    "http://localhost:5000/*"
  ]
}
```

This manifest file is similar to the one mentioned earlier, but it integrates smoothly with the `crxjs/vite-plugin` by bundling assets like **popup HTML**, **background script**, and other extension files using Vite.

#### 4. Modify Popup and Background Files
Place your **popup HTML** and **JavaScript** files inside the **frontend** folder, so they can be bundled and served by Vite.

##### `popup.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ChatGPT Templates</title>
  <link rel="stylesheet" href="./index.css" />
</head>
<body>
  <div id="app"></div>
  <script type="module" src="./src/main.tsx"></script>
</body>
</html>
```

The **popup** will be a React app, and you can build it just like your regular Vite-based React components.

##### `popup.js` (Using React):
```tsx
import React, { useEffect, useState } from 'react';

export const Popup = () => {
  const [ws, setWs] = useState<WebSocket | null>(null);
  const [response, setResponse] = useState<string | null>(null);

  useEffect(() => {
    const socket = new WebSocket('ws://localhost:5000');
    setWs(socket);

    socket.onopen = () => {
      console.log('WebSocket connected');
    };

    socket.onmessage = (event) => {
      setResponse(event.data);
    };

    return () => {
      socket.close();
    };
  }, []);

  const sendTemplate = (template: string) => {
    if (ws && ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify({ action: 'chatgpt_prompt', data: template }));
    }
  };

  return (
    <div>
      <h2>ChatGPT Templates</h2>
      <button onClick={() => sendTemplate('Template 1')}>
        Send Template 1
      </button>
      <button onClick={() => sendTemplate('Template 2')}>
        Send Template 2
      </button>
      <button onClick={() => sendTemplate('Template 3')}>
        Send Template 3
      </button>
      {response && <p>Response: {response}</p>}
    </div>
  );
};
```

In this example, `Popup.tsx` renders buttons that send predefined templates to your **WebSocket** backend and display the response.

#### 5. Background Script (`background.js`)
You can handle persistent tasks, WebSocket connections, or other logic inside a **background service worker**:

```js
// background.js
chrome.runtime.onInstalled.addListener(() => {
  console.log("ChatGPT Template Extension Installed");
});
```

This basic background script will log installation events, but you can expand it based on your requirements.

#### 6. Build and Run the Chrome Extension
To develop and build the extension:

1. **Development Mode**:
   Use the Vite development server while developing the extension:
   ```bash
   pnpm run dev
   ```
   This allows hot reloading of the Chrome extension popup.

2. **Build Mode**:
   To generate the production build of the Chrome extension:
   ```bash
   pnpm run build
   ```
   This command will output the built extension files in the `dist` directory, which you can load into Chrome.

#### 7. Load the Chrome Extension into Chrome
- Open **Chrome** and navigate to `chrome://extensions/`.
- Enable **Developer mode**.
- Click **Load unpacked** and select the **`dist`** folder generated by Vite.
- Your Chrome extension should now be available in your browser toolbar, and you can start interacting with it.

---

### Conclusion

By using **`crxjs/vite-plugin`** with Vite, you're able to:
- Use **Vite's fast development features** to build and serve a Chrome extension.
- **React** components for your popup or other UI features.
- Seamlessly integrate WebSocket connections between the extension and your local Node.js server.

This setup ensures that you can build modern Chrome extensions with the tools you're already using, while also making development faster and easier with hot-reloading and optimized builds. Let me know if you'd like further help with any part of the setup!
