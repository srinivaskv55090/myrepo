🔹 Step 1: Create a New Folder for the Extension

In VS Code (your sandbox), click File → Open Folder….

Create and open a folder called:

telemetry-checker

🔹 Step 2: Create package.json

Inside telemetry-checker, create a file named package.json with this content:

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

Inside the same folder, create extension.js and paste this:

const vscode = require('vscode');
const fetch = require('node-fetch'); // to call LLaMA3 API

async function callLlama3(code) {
    try {
        // 🔹 Replace with your real sandbox LLaMA3 endpoint
        const response = await fetch("http://localhost:8000/v1/completions", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                model: "llama3", 
                prompt: `Analyze this code and identify missing telemetry or observability (logging, tracing, OpenTelemetry, NewRelic):\n\n${code}`
            })
        });

        const result = await response.json();
        return result.choices[0].text;  // typical response format
    } catch (err) {
        return "Error calling LLaMA3: " + err.message;
    }
}

function activate(context) {
    let disposable = vscode.commands.registerCommand('telemetryChecker.analyzeCode', async function () {
        const editor = vscode.window.activeTextEditor;
        if (!editor) {
            vscode.window.showErrorMessage("No active editor!");
            return;
        }

        const code = editor.document.getText();
        vscode.window.showInformationMessage("Analyzing with LLaMA3...");

        const suggestions = await callLlama3(code);

        vscode.window.showInformationMessage("Telemetry Suggestions: " + suggestions);
    });

    context.subscriptions.push(disposable);
}

function deactivate() {}

module.exports = { activate, deactivate };

🔹 Step 4: Run the Extension

In VS Code menu → Run → Start Debugging (or press F5).

A new VS Code window opens: Extension Development Host.

Open any code file (Python, Java, etc.).

Press Ctrl+Shift+P → type Analyze Code for Telemetry → hit Enter.

It will send the file content to LLaMA3, and show suggestions like:

Telemetry Suggestions: Add OpenTelemetry span for HTTP call on line 12.

🔹 Step 5: Demo to Directors

Open sample code missing logs/metrics.

Run the command.

Show how AI points out missing telemetry and explains the impact.

⚠️ The only thing you must confirm:
👉 What is the API URL for LLaMA3 in your sandbox? (I used http://localhost:8000/v1/completions as an example.)

Do you see something like this in your sandbox documentation or model explorer?
