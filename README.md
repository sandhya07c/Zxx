# ⚡ Zx

**Zx is a local-first AI desktop companion and experimental AI command center** built with Python and PyQt6. It combines a desktop HUD, voice interaction through Google's Gemini API, memory, and a modular set of actions and plugins for working with files, browsers, desktop applications, web projects, Android devices, and Blender.

Zx explores more autonomous, multi-step AI workflows. It is an evolving research/development prototype—not a claim of real AGI or ASI. Capabilities vary with the configured model, API access, operating system, installed dependencies, connected devices, and enabled plugins.

## 🧠 What it does

- **Voice-first interaction:** a PyQt6 desktop interface with listening, thinking, speaking, and system-status feedback.
- **AI conversation:** Gemini-powered conversation and action selection.
- **Memory:** a memory manager for retrieving and storing useful context across sessions.
- **Actions and plugins:** modular tools for browser/computer work, file operations, research, media, and other tasks.
- **Application building:** an experimental app-builder that can generate a web project, install its dependencies, build it, and verify it with a browser when the needed tools are installed.
- **Optional integrations:** wireless Android control through a companion service, plus a local authenticated Blender bridge.
- **Interaction safeguards:** some sensitive or destructive operations use confirmation gates; review each action and its permissions.

## 🔄 How Zx works

```text
User (voice / interaction)
          |
          v
Zx Desktop UI (PyQt6 HUD)
          |
          v
Conversation Runtime (Gemini API)
          |
          v
Action & Plugin Discovery / Dispatch
          |
          +---- Memory and context
          +---- Browser / desktop / file tools
          +---- Web research / media / app builder
          +---- Optional Android companion bridge
          +---- Optional Blender bridge
          |
          v
Results, status updates, and assistant response
```

The action and plugin layers are modular. Optional integrations may require their own applications, credentials, companion services, permissions, or dependencies.

## 🚀 Quick start

### Requirements

- Git
- Python **3.11** recommended (the repository's CI workflow uses Python 3.11)
- A working microphone and speakers/headphones for voice interaction
- A Gemini API key for the Gemini-powered runtime
- Windows, macOS, or Linux; individual automation features may be platform-specific

### Windows (PowerShell)

```powershell
git clone https://github.com/chidvielasparepalli/Zx.git
cd Zx

py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
python -m pip install -r requirements.txt

python main.py
```

If PowerShell blocks environment activation, either enable scripts for your current user according to your device's policy or run the virtual-environment Python directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe main.py
```

### macOS / Linux

```bash
git clone https://github.com/chidvielasparepalli/Zx.git
cd Zx

python3.11 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -r requirements.txt

python main.py
```

### First-run configuration

The runtime reads the Gemini credential from `config/api_keys.json` using the `gemini_api_key` setting. Use the app's configuration flow if prompted, or add the key to the existing local configuration without deleting other settings:

```json
{
  "gemini_api_key": "YOUR_GEMINI_API_KEY"
}
```

Replace the placeholder with your key. **Never commit API keys, OAuth tokens, or other secrets.** Keep the real configuration local and do not paste secrets into issues or logs.

## 🧪 Verify the repository

Run these commands from the repository root with the virtual environment active:

```powershell
python -m compileall -q actions core tests main.py ui.py
python -m unittest discover -s tests -v
```

The checked-in unit tests currently cover app-builder utility functions. These commands are useful sanity checks but do not prove that every model, optional integration, device, or UI workflow works on every machine.

## 🧩 Project layout

```text
Zx/
├── actions/               # Callable actions and task helpers
├── core/                  # Runtime, model clients, orchestration, TTS/STT
├── config/                # Local runtime configuration and media assets
├── memory/                # Persistent memory manager
├── plugins/               # Optional tools and external integrations
├── Personality/           # Personality profiles and loader
├── blender_addon/         # Blender bridge, installer, and checks
├── dashboard/             # Optional remote dashboard
├── tests/                 # Unit tests
├── main.py                # Application entry point
├── ui.py                  # PyQt6 interface
├── requirements.txt       # Python dependencies
└── README.md               # Main project guide
```

## 🔌 Optional integrations and configuration

### App builder

The action name is `app_builder`. Its supported operations are `start`, `resume`, `status`, and `cancel`. Generated projects are created outside this repository; build verification may use Node.js/npm and Playwright/Chromium.

Optional environment settings:
- `ZX_BUILDER_MODEL` — model used by the app builder (default: `gemini-2.5-flash`).
- `ZX_BUILDER_PORT` — local preview port (default: `3180`).
- `GEMINI_API_KEY` — can supply the Gemini key to the builder when configured that way.

Builder state and logs are stored under the generated project's `.zx/` directory.

### Personality voice IDs

The optional ElevenLabs voices can be configured with `ZX_VOICE_WEDNESDAY`, `ZX_VOICE_TONY`, and `ZX_VOICE_DEADPOOL`. These are optional and only relevant when using those voice profiles with ElevenLabs configured.

### Android control

Android control requires the companion service on the phone and the desktop/phone to be able to reach each other over the configured local network or Tailscale connection. Follow the plugin's setup and permission requirements before use.

### Blender / 3D control

Blender integration uses a local HTTP bridge with an HMAC signature. See [the Blender integration guide](docs/BLENDER_INTEGRATION.md) for installation, configuration, protocol, and troubleshooting. Keep the bridge bound to localhost unless you deliberately configure a secured network setup.

## 🛡️ Safety and limitations

- Confirm permissions and review requests before enabling desktop, browser, or phone automation.
- Do not bypass CAPTCHA, OTP, MFA, passwords, or access controls.
- Use confirmation gates for sensitive, destructive, or irreversible operations.
- Generated code and automated actions can fail; inspect their results instead of assuming success.
- Keep secrets out of source control.
- Treat long-horizon execution as experimental and supervised.

## 📌 Project status

Zx is actively evolving. Some components are reusable utilities; others are prototypes that need real-environment testing. Build capabilities incrementally, verify results, and keep the user in control of sensitive decisions.
