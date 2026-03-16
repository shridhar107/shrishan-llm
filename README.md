# This Project is for using the LLMs effeciently

## Local LLMs

| Component | Improvement | Result |
|---|---|---|
| Embeddings | nomic-embed-text | Enables @codebase semantic search. |
| Architect | deepseek-r1:14b (98304) | Handles project-wide structural planning. |
| Developer | qwen2.5-coder:14b-instruct-q4_K_M (49152) | Delivers high-precision logic for complex tasks. |
| Roo Code | Agent Mode | Automates the "Write -> Test -> Fix" loop. |

### Provider Macbook

Install ollama
```bash
brew install ollama
```

```bash
which ollama
/opt/homebrew/bin/ollama
```

Start the Service effeciently
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://apple.com">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.ollama.service</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/ollama</string>
        <string>serve</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
        <key>OLLAMA_HOST</key>
        <string>0.0.0.0</string>
        <key>OLLAMA_MAX_LOADED_MODELS</key>
        <string>1</string>
        <key>OLLAMA_NUM_PARALLEL</key>
        <string>1</string>
        <key>OLLAMA_KEEP_ALIVE</key>
        <string>24h</string>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/ollama.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/ollama.err</string>
</dict>
</plist>
```

```bash
launchctl load ~/Library/LaunchAgents/com.ollama.service.plist
```

Stop the Service
```bash
launchctl unload ~/Library/LaunchAgents/com.ollama.service.plist
```

How to pull the Models ?
```bash
ollama pull deepseek-r1:14b
ollama pull qwen2.5-coder:14b-instruct-q4_K_M
ollama pull nomic-embed-text
```

How to make this local config as close to Claude Sonnet 4.6 as possible ?

```bash
cat <<EOF > Modelfile-architect
FROM deepseek-r1:14b
TEMPLATE """{{ if .System }}<｜System｜>{{ .System }}{{ end }}{{ if .Prompt }}<｜User｜>{{ .Prompt }}{{ end }}<｜Assistant｜>{{ if .Thought }}<｜Thought｜>{{ .Thought }}{{ end }}{{ .Response }}"""
PARAMETER num_ctx 98304
PARAMETER temperature 0.6
SYSTEM "You are an Elite Software Architect. Use your <think> block to deeply analyze the system architecture and cross-file dependencies before answering. Focus on patterns, potential regressions, and clean code."
EOF

ollama create architect-pro -f Modelfile-architect && rm Modelfile-architect


cat <<EOF > Modelfile-developer
FROM qwen2.5-coder:14b-instruct-q4_K_M
TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
{{ end }}<|im_start|>assistant
{{ .Response }}<|im_end|>"""
PARAMETER num_ctx 49152
PARAMETER temperature 0.0
SYSTEM "You are a world-class Senior Developer. Focus on writing clean, bug-free code. Be precise."
EOF

ollama create developer-pro -f Modelfile-developer && rm Modelfile-developer




```

Setup Reverse Proxy as inbound connection was not allowed on Macbook-1

```bash
# Syntax: ssh -N -R [RemotePort]:localhost:[LocalPort] [User]@[MacBook-2-IP]
ssh -N -R 11434:localhost:11434 shridhar@192.168.1.20
```
```bash
➜  fo-magician-v2 git:(dev/build-processing) ✗ lsof -i :11434
COMMAND     PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Code\x20H 37705 shridhar   71u  IPv4  0xcb583caef427d6d      0t0  TCP localhost:50152->localhost:11434 (ESTABLISHED)
sshd-sess 68442 shridhar   12u  IPv6 0x9dbf99901a80ce40      0t0  TCP localhost:11434 (LISTEN)
sshd-sess 68442 shridhar   14u  IPv4 0x6638f12fffa83ffe      0t0  TCP localhost:11434 (LISTEN)
sshd-sess 68442 shridhar   16u  IPv4 0x20b6f4f8e4593c65      0t0  TCP localhost:11434->localhost:50152 (ESTABLISHED)
➜  fo-magician-v2 git:(dev/build-processing) ✗
```

### User Macbook

#### VScode with Roo Code

Provider Selection
* 3rd-party Provider
* API Provider: Ollama
* Base URL: http://localhost:11434
* Ollama API Key:local(it can be anything)

We have created SSH reverse proxy tunnel, hence http://localhost:11434

Here is a condensed Roo Code Cheatsheet tailored for your Dual-MacBook setup (M2 32GB Host). Save this as ROO_CHEATSHEET.md in your project for quick reference.
------------------------------
🦘 Roo Code Agentic Cheatsheet🧠 Context Mentions (The "How It Sees")
Use these to feed your 98k Context Architect the right information from your codebase.

| Trigger | Command | Use Case |
|---|---|---|
| Project Map | @codebase | Search entire project using your local nomic-embed-text index. |
| Specific File | @path/to/file.ts | Pins a specific file for editing or reference. |
| Folder | @src/components/ | Adds all files in a folder to the current context. |
| Terminal | @terminal | Pastes the last 50 lines of your terminal (errors/logs). |
| Web Docs | @browser [url] | Scrapes live 2026 documentation to bypass model training cuts. |

------------------------------
🛠️ The "Claude-Like" Workflow (3-Step Loop)
To get Sonnet 4.6 quality, never ask for code immediately. Follow this cycle:
1. Research Phase (Use architect-pro)

Prompt: "Using @codebase, research how to implement [Feature X]. List all affected files and potential breaking changes. Do not write code yet."

2. Planning Phase (Use architect-pro)

Prompt: "Create a step-by-step implementation plan. List the exact functions that need refactoring. Wait for my approval."

3. Execution Phase (Switch to developer-pro)

Prompt: "Proceed with the plan. After each file change, run 'npm test' or 'npm run build'. If you see an error, fix it automatically before moving to the next file."

------------------------------
⚡ Agentic Commands
Roo Code is an Agent, not a chat. Use "Do" verbs:

* "Analyze...": Deep dive into logic without changing anything.
* "Refactor...": Improve structure while maintaining behavior.
* "Fix...": Find and squash bugs based on a terminal error.
* "Document...": Generate JSDoc/Readme based on existing code.

------------------------------
⌨️ Essential Keyboard Shortcuts

* Cmd + L: Open Roo Code Sidebar.
* Cmd + Enter: Send prompt / Approve plan.
* Esc: Stop generation (if the model "loops" or hallucinates).
* Cmd + Shift + P: Search "Roo Code: Toggle Mode" (to swap Architect/Developer).

------------------------------
🩺 MacBook-1 (Host) Health Check
Run these on your M2 32GB Mac if things feel slow:

* ollama ps: Verify only one model is loaded in the GPU.
* tail -f /tmp/ollama.err: Watch the 98k context loading in real-time.
* top -u: Check if the GPU/CPU is thermal throttling.

------------------------------
🚀 Pro-Tip for 32GB RAM
If a task is extremely complex, tell Roo:

"Think in a block first. Critically analyze three different approaches before picking the best one."

This forces the local model to simulate the "Chain of Thought" reasoning that makes Claude 4.6 feel superior.
------------------------------
Next Step: Ready to start your first session? Try asking: "@codebase find the three most complex files and explain why they are difficult to maintain."


