To integrate a **Google Chrome extension** with the **ChatGPT web interface** while leveraging your predefined local development server (which uses WebSocket for communication), you can create a Chrome extension that offers a popup interface with predefined templates. The popup can allow users to interact with your local server running on Node.js, React, and WebSocket, sending queries or templates that trigger specific interactions with ChatGPT.

### Step-by-Step Guide for Building the Chrome Extension with WebSocket Integration

#### 1. Chrome Extension Structure
A basic Chrome extension consists of the following files:
- `manifest.json`: Configuration file for your extension.
- `popup.html`: The HTML file for the popup window where the user interacts.
- `popup.js`: JavaScript file handling the behavior of the popup.
- `background.js`: A background script (optional) for handling persistent tasks.
- `content.js`: A script that interacts with the web page (optional if interacting directly with the local server).

#### 2. Manifest File (`manifest.json`)
The `manifest.json` describes the permissions and settings for your Chrome extension. Here is an example configuration:

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

#### 3. HTML Popup (`popup.html`)
This file contains the UI for the Chrome extension's popup, where users can select predefined templates to send to the local server.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ChatGPT Templates</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 10px;
            width: 300px;
        }
        button {
            margin: 5px 0;
            padding: 10px;
            width: 100%;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <h2>ChatGPT Templates</h2>
    <button id="template1">Template 1</button>
    <button id="template2">Template 2</button>
    <button id="template3">Template 3</button>

    <script src="popup.js"></script>
</body>
</html>
```

#### 4. Popup JavaScript (`popup.js`)
This script handles user interactions in the popup and sends the selected template via WebSocket to the local server.

```javascript
document.addEventListener('DOMContentLoaded', function () {
    const ws = new WebSocket('ws://localhost:5000'); // Connect to the local WebSocket server

    ws.onopen = () => {
        console.log("Connected to local WebSocket server");
    };

    ws.onmessage = (event) => {
        console.log("Received response from server: ", event.data);
        // Handle response from the server if necessary (e.g., display to the user)
    };

    ws.onerror = (error) => {
        console.error("WebSocket error: ", error);
    };

    document.getElementById('template1').addEventListener('click', () => {
        sendTemplate("Template 1: Example prompt for ChatGPT");
    });

    document.getElementById('template2').addEventListener('click', () => {
        sendTemplate("Template 2: Another ChatGPT interaction.");
    });

    document.getElementById('template3').addEventListener('click', () => {
        sendTemplate("Template 3: A third prompt for ChatGPT.");
    });

    function sendTemplate(template) {
        if (ws.readyState === WebSocket.OPEN) {
            ws.send(JSON.stringify({ action: "chatgpt_prompt", data: template }));
            console.log("Sent template to server: ", template);
        } else {
            console.error("WebSocket connection not open");
        }
    }
});
```

#### 5. Local WebSocket Server (Express + WebSocket in Node.js)
To connect with the Chrome extension, you need a WebSocket server on your local Node.js backend. Here's how to set up a WebSocket server using Express and `ws`:

- Install `ws` (WebSocket library for Node.js):
    ```bash
    pnpm install ws
    ```

- Add WebSocket integration to your Express backend (`backend/src/index.ts`):

```ts
import express from 'express';
import mongoose from 'mongoose';
import cors from 'cors';
import { WebSocketServer } from 'ws';

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
    useNewUrlParser: true,
    useUnifiedTopology: true
})
    .then(() => console.log('Connected to MongoDB'))
    .catch(err => console.error('MongoDB connection error:', err));

// Basic Route
app.get('/', (req, res) => {
    res.send('API is running');
});

// Start the server
const server = app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

// Setup WebSocket server
const wss = new WebSocketServer({ server });

wss.on('connection', (ws) => {
    console.log('WebSocket connection established');

    ws.on('message', (message) => {
        const data = JSON.parse(message.toString());
        console.log("Received: ", data);

        if (data.action === "chatgpt_prompt") {
            // Here, you'd send the prompt to ChatGPT via API or interact with your logic.
            console.log("Processing template: ", data.data);
            // Simulate response from ChatGPT
            ws.send(`ChatGPT Response to "${data.data}"`);
        }
    });

    ws.on('close', () => {
        console.log('WebSocket connection closed');
    });
});
```

### 6. Testing the Extension

1. **Local Server**:
   - Run your Node.js server:
     ```bash
     cd backend
     pnpm run dev
     ```

   Ensure your WebSocket server is running on `localhost:5000`.

2. **Load the Chrome Extension**:
   - Open **Chrome** and navigate to `chrome://extensions/`.
   - Enable **Developer mode**.
   - Click **Load unpacked** and select the folder where your Chrome extension files are located.

3. **Use the Extension**:
   - Click on the extension icon in the toolbar, and you will see the predefined templates in the popup.
   - Select a template to send it to the local server via WebSocket. The local server will process the prompt and respond (this can be displayed in the console or as a future extension improvement, you can show it in the popup).

---

### 7. Summary
You now have a Chrome extension that:
- Provides a popup for users to select predefined templates.
- Connects to a local WebSocket server via the Node.js backend.
- Sends data (templates) to the local server, which can then process it, e.g., by interacting with ChatGPT or some other service.

This setup allows the extension to act as an interface between the user and the local server, making interactions with ChatGPT seamless and customizable. You can expand this further by improving the UI/UX, handling additional actions, or adding more sophisticated logic for processing templates. Let me know if you'd like to dive deeper into any part!
