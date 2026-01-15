# Mool - OpenLoom Video Viewer

A minimal video viewer for OpenLoom recordings stored in Google Drive.

## Smart Naming Setup

OpenLoom can automatically generate descriptive filenames for recordings using local AI (fully offline).

### Requirements

- **mlx-whisper** - Apple Silicon optimized speech-to-text
- **LMStudio** - Local LLM server

### Installation

```bash
# Install mlx-whisper
brew install pipx
pipx install mlx-whisper

# Create the whisper API server
mkdir -p ~/.local/bin
curl -o ~/.local/bin/whisper-api-server.py \
  https://raw.githubusercontent.com/arnaudbreton/openloom/main/scripts/whisper-api-server.py
chmod +x ~/.local/bin/whisper-api-server.py

# Create launchd service (auto-starts on login)
cat > ~/Library/LaunchAgents/com.whisper.server.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.whisper.server</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/python3</string>
        <string>~/.local/bin/whisper-api-server.py</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/whisper-server.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/whisper-server.err</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:~/.local/bin</string>
    </dict>
</dict>
</plist>
EOF

# Start the service
launchctl load ~/Library/LaunchAgents/com.whisper.server.plist
```

### LMStudio Setup

1. Download [LMStudio](https://lmstudio.ai)
2. Load a model (recommended: `mistralai/ministral-3-3b`)
3. Start the local server (runs on `localhost:1234`)

### Enable in OpenLoom

1. Open the OpenLoom extension popup
2. Go to Settings
3. Scroll to "Smart Naming (Local AI)"
4. Enable and configure:
   - **Whisper Endpoint**: `http://localhost:8080`
   - **LLM Endpoint**: `http://localhost:1234`
   - **Model Name**: `mistralai/ministral-3-3b`

### How It Works

```
Recording → Whisper (transcribe) → LLM (summarize) → Descriptive Filename
```

Example output: `React_Component_Tutorial_2025-01-15.webm`

### Service Commands

```bash
# Check status
curl http://localhost:8080/health

# View logs
tail -f /tmp/whisper-server.log

# Restart
launchctl kickstart -k gui/$(id -u)/com.whisper.server

# Stop
launchctl unload ~/Library/LaunchAgents/com.whisper.server.plist
```
