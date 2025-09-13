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
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "telemetryChecker.analyzeCode",
        "title": "Analyze Code for Telemetry"
      }
    ]
  },
  "scripts": {
    "compile": "tsc -p ./"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0",
    "vscode": "^1.85.0"
  }
}
