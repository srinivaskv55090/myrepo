🔹 Step 2: Create package.json

In the Explorer (left panel), right-click → New File.

Name it: package.json.

Paste this content:

{
  "name": "telemetry-checker",
  "displayName": "Telemetry Checker",
  "description": "Analyzes code for missing telemetry/observability using LLaMA3.",
  "version": "0.0.1",
  "engines": {
    "vscode": "^1.85.0"
  },
  "activationEvents": [
    "onCommand:telemetryChecker.analyzeCode"
  ],
  "main": "./extension.js",
  "contributes": {
    "commands": [
      {
        "command": "telemetryChecker.analyzeCode",
        "title": "Analyze Code for Telemetry"
      }
    ]
  }
}

🔹 Step 3: Create extension.js

In the Explorer, right-click → New File.

Name it: extension.js.

Paste this code:

const vscode = require('vscode');
const http = require('http'); // use http if sandbox exposes llama3 locally
const https = require('https'); // use https if endpoint is secure

function activate(context) {
    let disposable = vscode.commands.registerCommand('telemetryChecker.analyzeCode', async function () {
        const editor = vscode.window.activeTextEditor;
        if (!editor) {
            vscode.window.showErrorMessage("No active editor!");
            return;
        }

        const code = editor.document.getText();

        // 🔹 Replace this with your real sandbox endpoint + port
        const data = JSON.stringify({ code });

        const options = {
            hostname: 'localhost',    // change if sandbox uses different host
            port: 8000,               // change if sandbox runs llama3 on another port
            path: '/llama3/analyze',  // change if API path differs
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Content-Length': data.length
            }
        };

        const req = http.request(options, res => {
            let body = '';
            res.on('data', chunk => { body += chunk; });
            res.on('end', () => {
                try {
                    const result = JSON.parse(body);
                    vscode.window.showInformationMessage(`Telemetry Suggestions: ${result.suggestions}`);
                } catch (err) {
                    vscode.window.showErrorMessage("Error parsing response: " + err);
                }
            });
        });

        req.on('error', error => {
            vscode.window.showErrorMessage("Request failed: " + error.message);
        });

        req.write(data);
        req.end();
    });

    context.subscriptions.push(disposable);
}

function deactivate() {}

module.exports = { activate, deactivate };

🔹 Step 4: Run the Extension

Go to the top menu: Run → Start Debugging (or press F5).

VS Code will open a new window called Extension Development Host.

In that window:

Open any code file (Python, Java, etc.).

Press Ctrl+Shift+P.

Search for Analyze Code for Telemetry.

Run it.

The extension will send your code to the LLaMA3 API and show suggestions.

🔹 Step 5: Demo Flow

Show some code missing telemetry (e.g., an API call without logging).

Run your command.

Extension shows:

Telemetry Suggestions: Add OpenTelemetry span here to trace request.


Explain impact (e.g., “Without telemetry, this won’t appear in distributed tracing”).

✅ Now you have:

One folder: telemetry-checker

Inside it:

package.json

extension.js

No npm install needed.
