# ObservabilityGuard VS Code Extension - Complete Development Guide

## Table of Contents
1. [Prerequisites & Environment Setup](#prerequisites--environment-setup)
2. [Project Structure](#project-structure)
3. [Core Extension Development](#core-extension-development)
4. [AI Integration with GitHub Copilot API](#ai-integration-with-github-copilot-api)
5. [Observability Detection Logic](#observability-detection-logic)
6. [Testing & Validation](#testing--validation)
7. [Packaging & Distribution](#packaging--distribution)
8. [Enterprise Deployment](#enterprise-deployment)

## Prerequisites & Environment Setup

### Mac Development Environment Setup

1. **Install Homebrew** (Package manager for Mac)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

2. **Install Node.js and npm**
```bash
brew install node
node --version  # Should be 18.x or higher
npm --version
```

3. **Install Visual Studio Code**
```bash
brew install --cask visual-studio-code
```

4. **Install VS Code Extension Development Tools**
```bash
npm install -g yo generator-code vsce
```

5. **Install Git**
```bash
brew install git
git --version
```

### Required VS Code Extensions
- GitHub Copilot (for AI integration)
- TypeScript and JavaScript Language Features

## Project Structure

Create the extension scaffold:

```bash
# Create project directory
mkdir observability-guard-extension
cd observability-guard-extension

# Generate extension template
yo code

# Choose:
# ? What type of extension do you want to create? New Extension (TypeScript)
# ? What's the name of your extension? ObservabilityGuard
# ? What's the identifier of your extension? observability-guard
# ? What's the description of your extension? AI-powered observability and telemetry validation for enterprise code
# ? Initialize a git repository? Yes
# ? Bundle the source code with webpack? Yes
# ? Package manager to use? npm
```

Final project structure:
```
observability-guard-extension/
├── package.json
├── tsconfig.json
├── webpack.config.js
├── .vscode/
├── src/
│   ├── extension.ts
│   ├── analyzers/
│   │   ├── observabilityAnalyzer.ts
│   │   ├── telemetryAnalyzer.ts
│   │   └── loggingAnalyzer.ts
│   ├── ai/
│   │   ├── copilotIntegration.ts
│   │   └── promptTemplates.ts
│   ├── diagnostics/
│   │   └── diagnosticProvider.ts
│   ├── config/
│   │   └── observabilityRules.ts
│   └── utils/
│       └── codeParser.ts
├── resources/
├── test/
└── README.md
```

## Core Extension Development

### 1. Package.json Configuration

```json
{
  "name": "observability-guard",
  "displayName": "ObservabilityGuard",
  "description": "AI-powered observability and telemetry validation for enterprise code",
  "version": "1.0.0",
  "publisher": "capital-one-sre",
  "engines": {
    "vscode": "^1.80.0"
  },
  "categories": ["Linters", "Other"],
  "keywords": ["observability", "telemetry", "opentelemetry", "monitoring", "sre"],
  "activationEvents": [
    "onLanguage:javascript",
    "onLanguage:typescript",
    "onLanguage:python",
    "onLanguage:java",
    "onLanguage:go",
    "onLanguage:csharp"
  ],
  "main": "./dist/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "observabilityGuard.analyzeFile",
        "title": "Analyze Current File for Observability",
        "category": "ObservabilityGuard"
      },
      {
        "command": "observabilityGuard.analyzeWorkspace",
        "title": "Analyze Entire Workspace",
        "category": "ObservabilityGuard"
      },
      {
        "command": "observabilityGuard.generateTelemetry",
        "title": "Generate Missing Telemetry Code",
        "category": "ObservabilityGuard"
      }
    ],
    "configuration": {
      "title": "ObservabilityGuard",
      "properties": {
        "observabilityGuard.enableRealTimeAnalysis": {
          "type": "boolean",
          "default": true,
          "description": "Enable real-time code analysis"
        },
        "observabilityGuard.strictMode": {
          "type": "boolean",
          "default": false,
          "description": "Enable strict observability requirements"
        },
        "observabilityGuard.supportedFrameworks": {
          "type": "array",
          "default": ["express", "fastify", "spring-boot", "gin", "flask"],
          "description": "Supported web frameworks for analysis"
        }
      }
    },
    "problemMatchers": [
      {
        "name": "observability-issues",
        "owner": "observability-guard",
        "fileLocation": ["relative", "${workspaceFolder}"],
        "pattern": {
          "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error|info):\\s+(.*)$",
          "file": 1,
          "line": 2,
          "column": 3,
          "severity": 4,
          "message": 5
        }
      }
    ]
  },
  "scripts": {
    "vscode:prepublish": "npm run package",
    "compile": "webpack",
    "watch": "webpack --watch",
    "package": "webpack --mode production --devtool hidden-source-map",
    "test": "node ./out/test/runTest.js"
  },
  "devDependencies": {
    "@types/vscode": "^1.80.0",
    "@types/node": "16.x",
    "typescript": "^5.1.6",
    "webpack": "^5.88.1",
    "webpack-cli": "^5.1.4",
    "ts-loader": "^9.4.4"
  },
  "dependencies": {
    "@octokit/rest": "^20.0.1",
    "axios": "^1.5.0",
    "tree-sitter": "^0.20.4",
    "tree-sitter-javascript": "^0.20.2",
    "tree-sitter-typescript": "^0.20.3",
    "tree-sitter-python": "^0.20.4",
    "tree-sitter-java": "^0.20.2"
  }
}
```

### 2. Main Extension File (src/extension.ts)

```typescript
import * as vscode from 'vscode';
import { ObservabilityAnalyzer } from './analyzers/observabilityAnalyzer';
import { DiagnosticProvider } from './diagnostics/diagnosticProvider';
import { CopilotIntegration } from './ai/copilotIntegration';

let diagnosticCollection: vscode.DiagnosticCollection;
let analyzer: ObservabilityAnalyzer;
let diagnosticProvider: DiagnosticProvider;
let copilotIntegration: CopilotIntegration;

export function activate(context: vscode.ExtensionContext) {
    console.log('ObservabilityGuard extension is now active!');

    // Initialize components
    diagnosticCollection = vscode.languages.createDiagnosticCollection('observability-guard');
    analyzer = new ObservabilityAnalyzer();
    diagnosticProvider = new DiagnosticProvider(diagnosticCollection);
    copilotIntegration = new CopilotIntegration();

    // Register commands
    const analyzeFileCommand = vscode.commands.registerCommand(
        'observabilityGuard.analyzeFile',
        analyzeCurrentFile
    );

    const analyzeWorkspaceCommand = vscode.commands.registerCommand(
        'observabilityGuard.analyzeWorkspace',
        analyzeWorkspace
    );

    const generateTelemetryCommand = vscode.commands.registerCommand(
        'observabilityGuard.generateTelemetry',
        generateTelemetryCode
    );

    // Register text document change listener for real-time analysis
    const documentChangeListener = vscode.workspace.onDidChangeTextDocument(
        (event) => {
            if (isRealTimeAnalysisEnabled() && isSupportedLanguage(event.document.languageId)) {
                debounceAnalyze(event.document);
            }
        }
    );

    // Register document save listener
    const documentSaveListener = vscode.workspace.onDidSaveTextDocument(
        (document) => {
            if (isSupportedLanguage(document.languageId)) {
                analyzeDocument(document);
            }
        }
    );

    // Register document open listener
    const documentOpenListener = vscode.workspace.onDidOpenTextDocument(
        (document) => {
            if (isSupportedLanguage(document.languageId)) {
                analyzeDocument(document);
            }
        }
    );

    context.subscriptions.push(
        analyzeFileCommand,
        analyzeWorkspaceCommand,
        generateTelemetryCommand,
        documentChangeListener,
        documentSaveListener,
        documentOpenListener,
        diagnosticCollection
    );

    // Show welcome message
    showWelcomeMessage();
}

async function analyzeCurrentFile() {
    const editor = vscode.window.activeTextEditor;
    if (!editor) {
        vscode.window.showWarningMessage('No active file to analyze');
        return;
    }

    await analyzeDocument(editor.document);
    vscode.window.showInformationMessage('Observability analysis completed');
}

async function analyzeWorkspace() {
    const workspaceFolders = vscode.workspace.workspaceFolders;
    if (!workspaceFolders) {
        vscode.window.showWarningMessage('No workspace folder found');
        return;
    }

    vscode.window.withProgress({
        location: vscode.ProgressLocation.Notification,
        title: 'Analyzing workspace for observability issues...',
        cancellable: true
    }, async (progress, token) => {
        const files = await vscode.workspace.findFiles('**/*.{js,ts,py,java,go,cs}', '**/node_modules/**');
        const totalFiles = files.length;

        for (let i = 0; i < files.length; i++) {
            if (token.isCancellationRequested) {
                break;
            }

            const document = await vscode.workspace.openTextDocument(files[i]);
            await analyzeDocument(document);
            
            progress.report({ 
                increment: (100 / totalFiles),
                message: `Analyzed ${i + 1}/${totalFiles} files`
            });
        }
    });
}

async function generateTelemetryCode() {
    const editor = vscode.window.activeTextEditor;
    if (!editor) {
        vscode.window.showWarningMessage('No active file');
        return;
    }

    const document = editor.document;
    const issues = await analyzer.analyzeDocument(document);
    
    if (issues.length === 0) {
        vscode.window.showInformationMessage('No observability issues found!');
        return;
    }

    // Use AI to generate fixes
    const fixes = await copilotIntegration.generateTelemetryFixes(document, issues);
    
    // Apply fixes
    const edit = new vscode.WorkspaceEdit();
    fixes.forEach(fix => {
        edit.insert(document.uri, fix.position, fix.code);
    });

    await vscode.workspace.applyEdit(edit);
    vscode.window.showInformationMessage(`Generated ${fixes.length} telemetry improvements`);
}

async function analyzeDocument(document: vscode.TextDocument) {
    try {
        const issues = await analyzer.analyzeDocument(document);
        diagnosticProvider.updateDiagnostics(document.uri, issues);
    } catch (error) {
        console.error('Analysis failed:', error);
        vscode.window.showErrorMessage(`Analysis failed: ${error}`);
    }
}

// Debounce function to avoid excessive analysis during typing
let debounceTimer: NodeJS.Timeout | undefined;
function debounceAnalyze(document: vscode.TextDocument) {
    if (debounceTimer) {
        clearTimeout(debounceTimer);
    }
    
    debounceTimer = setTimeout(() => {
        analyzeDocument(document);
    }, 1000); // 1 second delay
}

function isSupportedLanguage(languageId: string): boolean {
    const supportedLanguages = ['javascript', 'typescript', 'python', 'java', 'go', 'csharp'];
    return supportedLanguages.includes(languageId);
}

function isRealTimeAnalysisEnabled(): boolean {
    const config = vscode.workspace.getConfiguration('observabilityGuard');
    return config.get('enableRealTimeAnalysis', true);
}

function showWelcomeMessage() {
    const config = vscode.workspace.getConfiguration('observabilityGuard');
    const hasShownWelcome = context.globalState.get('hasShownWelcome', false);
    
    if (!hasShownWelcome) {
        vscode.window.showInformationMessage(
            'ObservabilityGuard is now active! It will help you maintain proper observability in your code.',
            'Learn More'
        ).then(selection => {
            if (selection === 'Learn More') {
                vscode.env.openExternal(vscode.Uri.parse('https://internal.capitalone.com/observability-guard'));
            }
        });
        
        context.globalState.update('hasShownWelcome', true);
    }
}

export function deactivate() {
    if (diagnosticCollection) {
        diagnosticCollection.dispose();
    }
}
```

## AI Integration with GitHub Copilot API

### 3. Copilot Integration (src/ai/copilotIntegration.ts)

```typescript
import * as vscode from 'vscode';
import { ObservabilityIssue } from '../analyzers/observabilityAnalyzer';
import { PromptTemplates } from './promptTemplates';

export interface TelemetryFix {
    position: vscode.Position;
    code: string;
    description: string;
}

export class CopilotIntegration {
    private promptTemplates: PromptTemplates;

    constructor() {
        this.promptTemplates = new PromptTemplates();
    }

    async generateTelemetryFixes(
        document: vscode.TextDocument, 
        issues: ObservabilityIssue[]
    ): Promise<TelemetryFix[]> {
        const fixes: TelemetryFix[] = [];

        for (const issue of issues) {
            try {
                const fix = await this.generateFixForIssue(document, issue);
                if (fix) {
                    fixes.push(fix);
                }
            } catch (error) {
                console.error(`Failed to generate fix for issue: ${issue.message}`, error);
            }
        }

        return fixes;
    }

    private async generateFixForIssue(
        document: vscode.TextDocument, 
        issue: ObservabilityIssue
    ): Promise<TelemetryFix | null> {
        const context = this.getCodeContext(document, issue.range);
        const prompt = this.promptTemplates.generateFixPrompt(issue, context);

        // Use GitHub Copilot Chat API
        const response = await this.callCopilotAPI(prompt);
        
        if (response && response.code) {
            return {
                position: issue.range.start,
                code: response.code,
                description: response.description || `Fix for ${issue.type}`
            };
        }

        return null;
    }

    private async callCopilotAPI(prompt: string): Promise<{code: string, description: string} | null> {
        try {
            // GitHub Copilot Chat API integration
            const copilotApi = await vscode.lm.requestLanguageModelAccess({
                vendor: 'copilot',
                family: 'gpt-4'
            });

            if (!copilotApi) {
                throw new Error('GitHub Copilot not available');
            }

            const messages = [
                vscode.lm.LanguageModelChatUserMessage.User(prompt)
            ];

            const response = await copilotApi.requestChatResponse(messages);
            
            let fullResponse = '';
            for await (const part of response.text) {
                fullResponse += part;
            }

            return this.parseAIResponse(fullResponse);
        } catch (error) {
            console.error('Copilot API call failed:', error);
            return null;
        }
    }

    private parseAIResponse(response: string): {code: string, description: string} | null {
        try {
            // Extract code blocks from the response
            const codeBlockRegex = /```[\w]*\n([\s\S]*?)\n```/g;
            const match = codeBlockRegex.exec(response);
            
            if (match && match[1]) {
                return {
                    code: match[1].trim(),
                    description: response.split('```')[0].trim()
                };
            }

            return null;
        } catch (error) {
            console.error('Failed to parse AI response:', error);
            return null;
        }
    }

    private getCodeContext(document: vscode.TextDocument, range: vscode.Range): string {
        const startLine = Math.max(0, range.start.line - 5);
        const endLine = Math.min(document.lineCount - 1, range.end.line + 5);
        
        let context = '';
        for (let i = startLine; i <= endLine; i++) {
            context += `${i + 1}: ${document.lineAt(i).text}\n`;
        }
        
        return context;
    }
}
```

### 4. AI Prompt Templates (src/ai/promptTemplates.ts)

```typescript
import { ObservabilityIssue, ObservabilityIssueType } from '../analyzers/observabilityAnalyzer';

export class PromptTemplates {
    generateFixPrompt(issue: ObservabilityIssue, codeContext: string): string {
        const basePrompt = `You are an expert Site Reliability Engineer at Capital One Bank. Your task is to add proper observability and telemetry to enterprise-grade code.

Code Context:
\`\`\`
${codeContext}
\`\`\`

Issue: ${issue.message}
Type: ${issue.type}
Severity: ${issue.severity}

Requirements:
- Follow enterprise security and compliance standards
- Use industry-standard observability libraries (OpenTelemetry preferred)
- Include proper error handling and logging
- Add metrics collection where appropriate
- Ensure distributed tracing is properly implemented
- Follow the principle of structured logging

Please provide the missing observability code that should be added to fix this issue. Include only the necessary code additions, with clear comments explaining the purpose.

Response format:
\`\`\`typescript
// Your observability code here
\`\`\`

Brief explanation of the fix.`;

        return basePrompt + this.getSpecificPromptForIssueType(issue.type);
    }

    private getSpecificPromptForIssueType(type: ObservabilityIssueType): string {
        switch (type) {
            case ObservabilityIssueType.MISSING_OPENTELEMETRY:
                return `

Specific requirements for OpenTelemetry:
- Initialize OpenTelemetry SDK with proper resource attributes
- Set up tracing provider with appropriate configuration
- Include service name, version, and environment
- Configure exporters for your observability backend
- Add automatic instrumentation for HTTP, database, and other frameworks`;

            case ObservabilityIssueType.MISSING_LOGGING:
                return `

Specific requirements for logging:
- Use structured logging (JSON format preferred)
- Include correlation IDs for request tracing
- Add appropriate log levels (INFO, WARN, ERROR)
- Include relevant context (user ID, request ID, etc.)
- Ensure sensitive data is not logged`;

            case ObservabilityIssueType.MISSING_METRICS:
                return `

Specific requirements for metrics:
- Create custom metrics for business-critical operations
- Include standard HTTP metrics (response time, status codes)
- Add counter, gauge, and histogram metrics as appropriate
- Use consistent naming conventions
- Include relevant labels/tags for filtering`;

            case ObservabilityIssueType.MISSING_ERROR_HANDLING:
                return `

Specific requirements for error handling:
- Catch and properly log all exceptions
- Include stack traces for debugging
- Add error metrics and alerting
- Implement circuit breaker patterns where appropriate
- Ensure errors are properly propagated in distributed systems`;

            case ObservabilityIssueType.MISSING_HEALTH_CHECK:
                return `

Specific requirements for health checks:
- Implement liveness and readiness probes
- Check database connectivity
- Verify external service dependencies
- Return appropriate HTTP status codes
- Include health check metrics`;

            default:
                return '';
        }
    }

    generateAnalysisPrompt(code: string, language: string): string {
        return `You are an expert Site Reliability Engineer reviewing ${language} code for observability issues.

Analyze this code and identify missing observability components:

\`\`\`${language}
${code}
\`\`\`

Look for:
1. Missing OpenTelemetry instrumentation
2. Insufficient logging statements
3. Missing metrics collection
4. Poor error handling
5. Missing health check endpoints
6. Missing distributed tracing
7. Security vulnerabilities in logging

For each issue found, provide:
- Issue type
- Severity (low, medium, high, critical)
- Line number (if applicable)
- Detailed description
- Recommended fix

Return results in JSON format:
\`\`\`json
{
  "issues": [
    {
      "type": "MISSING_OPENTELEMETRY",
      "severity": "high",
      "line": 15,
      "message": "OpenTelemetry tracing not initialized",
      "recommendation": "Add OpenTelemetry SDK initialization"
    }
  ]
}
\`\`\``;
    }
}
```

## Observability Detection Logic

### 5. Main Analyzer (src/analyzers/observabilityAnalyzer.ts)

```typescript
import * as vscode from 'vscode';
import { TelemetryAnalyzer } from './telemetryAnalyzer';
import { LoggingAnalyzer } from './loggingAnalyzer';
import { CodeParser } from '../utils/codeParser';

export enum ObservabilityIssueType {
    MISSING_OPENTELEMETRY = 'MISSING_OPENTELEMETRY',
    MISSING_LOGGING = 'MISSING_LOGGING',
    MISSING_METRICS = 'MISSING_METRICS',
    MISSING_ERROR_HANDLING = 'MISSING_ERROR_HANDLING',
    MISSING_HEALTH_CHECK = 'MISSING_HEALTH_CHECK',
    MISSING_TRACING = 'MISSING_TRACING',
    INSECURE_LOGGING = 'INSECURE_LOGGING',
    MISSING_STRUCTURED_LOGGING = 'MISSING_STRUCTURED_LOGGING'
}

export enum ObservabilitySeverity {
    LOW = 'low',
    MEDIUM = 'medium',
    HIGH = 'high',
    CRITICAL = 'critical'
}

export interface ObservabilityIssue {
    type: ObservabilityIssueType;
    severity: ObservabilitySeverity;
    message: string;
    range: vscode.Range;
    source: string;
    relatedInformation?: vscode.DiagnosticRelatedInformation[];
}

export class ObservabilityAnalyzer {
    private telemetryAnalyzer: TelemetryAnalyzer;
    private loggingAnalyzer: LoggingAnalyzer;
    private codeParser: CodeParser;

    constructor() {
        this.telemetryAnalyzer = new TelemetryAnalyzer();
        this.loggingAnalyzer = new LoggingAnalyzer();
        this.codeParser = new CodeParser();
    }

    async analyzeDocument(document: vscode.TextDocument): Promise<ObservabilityIssue[]> {
        const issues: ObservabilityIssue[] = [];
        const text = document.getText();
        const languageId = document.languageId;

        try {
            // Parse the code to understand its structure
            const ast = await this.codeParser.parseCode(text, languageId);
            
            // Run different analyzers
            const telemetryIssues = await this.telemetryAnalyzer.analyze(document, ast);
            const loggingIssues = await this.loggingAnalyzer.analyze(document, ast);
            const structuralIssues = await this.analyzeStructural(document, ast);
            const securityIssues = await this.analyzeSecurityConcerns(document, ast);

            issues.push(
                ...telemetryIssues,
                ...loggingIssues,
                ...structuralIssues,
                ...securityIssues
            );

            // Sort by severity
            issues.sort((a, b) => this.getSeverityWeight(b.severity) - this.getSeverityWeight(a.severity));

        } catch (error) {
            console.error('Analysis failed:', error);
            // Return a generic issue if parsing fails
            issues.push({
                type: ObservabilityIssueType.MISSING_OPENTELEMETRY,
                severity: ObservabilitySeverity.MEDIUM,
                message: 'Unable to analyze file completely. Please check for syntax errors.',
                range: new vscode.Range(0, 0, 0, 0),
                source: 'observability-guard'
            });
        }

        return issues;
    }

    private async analyzeStructural(
        document: vscode.TextDocument, 
        ast: any
    ): Promise<ObservabilityIssue[]> {
        const issues: ObservabilityIssue[] = [];
        const text = document.getText();

        // Check for HTTP endpoints without health checks
        if (this.hasHttpEndpoints(ast) && !this.hasHealthCheck(text)) {
            issues.push({
                type: ObservabilityIssueType.MISSING_HEALTH_CHECK,
                severity: ObservabilitySeverity.HIGH,
                message: 'HTTP service missing health check endpoint',
                range: new vscode.Range(0, 0, 0, 0),
                source: 'observability-guard'
            });
        }

        // Check for database operations without proper error handling
        const dbOperations = this.findDatabaseOperations(ast);
        for (const operation of dbOperations) {
            if (!this.hasProperErrorHandling(operation, ast)) {
                issues.push({
                    type: ObservabilityIssueType.MISSING_ERROR_HANDLING,
                    severity: ObservabilitySeverity.HIGH,
                    message: 'Database operation missing error handling and logging',
                    range: this.getRange(operation, document),
                    source: 'observability-guard'
                });
            }
        }

        // Check for async operations without proper tracing
        const asyncOperations = this.findAsyncOperations(ast);
        for (const operation of asyncOperations) {
            if (!this.hasTracing(operation, text)) {
                issues.push({
                    type: ObservabilityIssueType.MISSING_TRACING,
                    severity: ObservabilitySeverity.MEDIUM,
                    message: 'Async operation missing distributed tracing',
                    range: this.getRange(operation, document),
                    source: 'observability-guard'
                });
            }
        }

        return issues;
    }

    private async analyzeSecurityConcerns(
        document: vscode.TextDocument, 
        ast: any
    ): Promise<ObservabilityIssue[]> {
        const issues: ObservabilityIssue[] = [];
        const text = document.getText();

        // Check for potential PII in logging
        const logStatements = this.findLogStatements(ast);
        for (const logStatement of logStatements) {
            if (this.containsPotentialPII(logStatement, text)) {
                issues.push({
                    type: ObservabilityIssueType.INSECURE_LOGGING,
                    severity: ObservabilitySeverity.CRITICAL,
                    message: 'Potential PII or sensitive data in log statement',
                    range: this.getRange(logStatement, document),
                    source: 'observability-guard'
                });
            }
        }

        return issues;
    }

    // Helper methods for analysis
    private hasHttpEndpoints(ast: any): boolean {
        // Implementation depends on AST structure
        // Look for Express, FastAPI, Spring Boot endpoints, etc.
        return false; // Placeholder
    }

    private hasHealthCheck(text: string): boolean {
        const healthCheckPatterns = [
            /\/health/i,
            /\/healthcheck/i,
            /\/status/i,
            /health.*endpoint/i,
            /readiness.*probe/i,
            /liveness.*probe/i
        ];
        
        return healthCheckPatterns.some(pattern => pattern.test(text));
    }

    private findDatabaseOperations(ast: any): any[] {
        // Find database operations in AST
        return []; // Placeholder
    }

    private findAsyncOperations(ast: any): any[] {
        // Find async/await, promises, etc.
        return []; // Placeholder
    }

    private findLogStatements(ast: any): any[] {
        // Find console.log, logger.info, etc.
        return []; // Placeholder
    }

    private hasProperErrorHandling(operation: any, ast: any): boolean {
        // Check if operation is wrapped in try-catch or has error handling
        return false; // Placeholder
    }

    private hasTracing(operation: any, text: string): boolean {
        const tracingPatterns = [
            /trace/i,
            /span/i,
            /opentelemetry/i,
            /@trace/i
        ];
        
        return tracingPatterns.some(pattern => pattern.test(text));
    }

    private containsPotentialPII(logStatement: any, text: string): boolean {
        const piiPatterns = [
            /password/i,
            /ssn/i,
            /social.security/i,
            /credit.card/i,
            /email.*@/i,
            /phone.*\d{3}-\d{3}-\d{4}/i,
            /token/i,
            /api.key/i
        ];
        
        return piiPatterns.some(pattern => pattern.test(text));
    }

    private getRange(node: any, document: vscode.TextDocument): vscode.Range {
        // Convert AST node
