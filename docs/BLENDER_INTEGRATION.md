# Zx Blender Integration Guide

This guide covers the optional Blender bridge included in Zx. It uses a localhost HTTP service and signed JSON requests; it is separate from the main desktop app and requires Blender to be installed.

# 🧊 Zx Blender / 3D Integration

Zx can drive a local Blender instance to create and modify 3D scenes
through natural-language requests. This is a **native plugin capability**, not a
separate project and not a hardcoded model library: the AI agent plans each task,
executes general Blender operations step by step, inspects the scene after each
step, and corrects failures — exactly as a human modeller would.

### Architecture

```text
Zx AI Agent
   │   (perceives request → plans → selects the blender_control tool)
   ▼
blender_control plugin         plugins/blender_control.py
   │   (maps agent ops → bridge JSON, HMAC-signed)
   ▼
_bridge client helper          plugins/_blender_bridge_client.py
   │   (localhost HTTP + JSON, HMAC-SHA256 signature)
   ▼
Blender bridge add-on          blender_addon/blender_zx_bridge.py
   │   (runs INSIDE Blender, whitelisted command table, main-thread timer)
   ▼
Blender Python API (bpy)
   ▼
3D scene / model / materials / animation / render
```

### Why a dedicated bridge (not Blender MCP)

Blender MCP's add-on speaks a private WebSocket protocol tied to Claude Code's MCP
framework. Zx has no MCP stack and must not depend on Claude Code's
protocol. This bridge is a plain HTTP+JSON service on `127.0.0.1` using only the
Python standard library on both sides, so it works anywhere Blender runs and
stays fully local.

### Installation

1. **Install the Blender add-on**
   ```
   python blender_addon/install_blender_addon.py
   ```
   (copies the add-on into Blender's user scripts/addons and enables it, or
   just do it manually: Preferences → Add-ons → Install → pick
   `blender_addon/blender_zx_bridge.py` → enable.)

2. **Launch Blender** (GUI is recommended; headless also works — see below).
   When the add-on starts it prints:
   `Zx Bridge: ready on http://127.0.0.1:8147`

3. **Zx finds it automatically.** The add-on writes a one-time HMAC
   token into Blender's user `scripts/` dir; the plugin reads the same file, so
   no manual key handshake is needed. The bridge is bound to `127.0.0.1` only.

### Configuration

Everything lives in the plugin settings (⚙ → Plugin Settings → Blender / 3D):

| Field | Default | Meaning |
|---|---|---|
| bridge_host | 127.0.0.1 | Bridge address (keep loopback) |
| bridge_port | 8147 | Bridge port (NOT 9876 — that's Blender MCP's) |
| bridge_token | *(auto)* | HMAC token; blank = auto-pair from Blender's scripts dir |
| output_dir | *(OS temp)* | Default render output folder |
| confirm_destructive | true | Ask for on-screen confirmation before delete/clear/render |
| exec_python | false | Enable the execute_python command (expert only) |

Values persist in `config/api_keys.json` under `plugin_config.blender_control`.

### Supported operations

| Group | Ops |
|---|---|
| Connection | `status`, `health` |
| Scene inspection | `list_objects`, `scene_info` |
| Objects | `create_primitive`, `create_mesh`, `set_transform`, `duplicate_object`, `rename_object`, `delete_objects`, `clear_scene`, `object_collection`, `add_modifier`, `boolean_modifier` |
| Materials | `create_material`, `set_material`, `set_world` |
| Lighting | `create_light` |
| Cameras | `create_camera` (optional `aim_at`) |
| Animation | `add_keyframe`, `set_timeline` |
| Rendering | `set_render`, `render` |
| File | `save_blend`, `open_blend` |
| Advanced | `execute_python` (disabled by default) |

Primitives: cube, sphere, uvsphere, ico, cylinder, cone, torus, plane, grid,
circle, monkey, empty. Modifiers: subsurf, bevel, array, mirror, solidify,
boolean, decimate, remesh, and more.

### Security model

- The bridge binds to **127.0.0.1 only** — nothing external can reach it.
- Every `/rpc` call requires an HMAC-SHA256 signature; unauthenticated POSTs
  return 401. `execute_python` is **off by default** and stays off unless
  explicitly enabled in the add-on preferences.
- Render sample count is hard-capped at 256 by the bridge.
- Destructive/heavy operations (`delete_objects`, `clear_scene`, `save_blend`,
  `open_blend`, `render`, `execute_python`) require an **on-screen confirmation
  banner** (via the same `core/confirm.py` gate used by shutdown/restart) unless
  `confirm_destructive` is turned off.
- No CSRF surface: the HTTP handler rejects any non-`application/json`
  content-type and enforces `Content-Length`.

### Example commands (spoken)

> "Check if Blender is connected."
> "Create a futuristic spaceship."
> "Add a red metallic material to the Cube."
> "Add a point light and a camera aimed at the ship."
> "Render the scene and save it."

The agent plans multi-step tasks autonomously. For "create a spaceship" it will:
create primitives for the hull/wings/engines → transform them → apply metallic
materials → add a glowing engine material → add lights → place a camera → inspect
→ render.

### Headless (background) mode

The bridge serves two ways:
- **GUI Blender** — the add-on runs commands on Blender's main thread via a
  `bpy.app.timers` callback (fast, responsive).
- **Background (`blender -b --python blender_addon/headless_driver.py`)** — the
  driver loop drains the queue. Use this for automated rendering / CI.

### Troubleshooting

- **`bridge unreachable`** — Blender isn't running, the add-on isn't enabled, or
  the port is wrong. Enable the add-on and confirm the "ready on
  http://127.0.0.1:8147" line in the Blender console (or **Window → Toggle System
  Console**).
- **401 / token mismatch** — delete the stale `zx_bridge_token.txt` in
  Blender's `scripts/` dir, or set the same token in both places.
- **"Cannot render, no camera"** — create a camera and set the scene's active
  camera (`create_camera` with `aim_at` handles this automatically).
- **Blender MCP already on 9876** — keep the Zx bridge on 8147; no conflict.

For the add-on's own documentation see `blender_addon/README.md`.

🖥️ Computer Interaction

Future versions of Zx are intended to interact with the computer environment.

Potential capabilities include:

Application launching
File operations
Browser interaction
Keyboard input
Mouse interaction
Screen understanding
Terminal interaction
Code execution
Development workflows
Error diagnosis

Computer-control actions should operate inside defined permission and sandbox boundaries.

💻 Coding Intelligence

Zx is intended to become capable of assisting with complete software-development workflows.

Potential workflow:

Requirement
    ↓
Understand
    ↓
Architecture
    ↓
Implementation
    ↓
Run
    ↓
Test
    ↓
Observe Error
    ↓
Debug
    ↓
Fix
    ↓
Retest
    ↓
Validate

The objective is not merely code generation.

The system should eventually be able to understand a development objective, modify a project, execute tests, analyze failures, and iterate toward a verified result.

🌐 Web Research

A future research system can operate as:

Research Goal
     ↓
Search
     ↓
Collect Sources
     ↓
Extract Information
     ↓
Compare Evidence
     ↓
Reason
     ↓
Generate Findings
     ↓
Store Knowledge

The system should distinguish between:

Retrieved information
Model-generated reasoning
Assumptions
Unverified information
Verified results
📊 Data Analysis

Zx is intended to support general data workflows.

Dataset
   ↓
Inspect
   ↓
Clean
   ↓
Analyze
   ↓
Visualize
   ↓
Interpret
   ↓
Generate Findings

Potential applications include:

CSV analysis
Spreadsheet analysis
Statistical analysis
Data visualization
Pattern detection
Report generation
Automated insights
🧪 Experimentation & Hypothesis Generation

An advanced research capability could allow Zx to generate and test hypotheses.

Observation
     ↓
Question
     ↓
Hypothesis
     ↓
Experiment Design
     ↓
Execute
     ↓
Collect Results
     ↓
Evaluate
     ↓
Update Knowledge

This is intended as an experimental research direction rather than a claim of autonomous scientific discovery.

🛡️ Safety Architecture

Autonomy must be combined with strong safety boundaries.

The planned architecture is:

User Request
     ↓
Intent Analysis
     ↓
Risk Assessment
     ↓
Permission Check
     ↓
Tool Selection
     ↓
Execution
     ↓
Result Verification
     ↓
User Confirmation

High-impact actions should require explicit approval.

Examples include:

Sending important communications
Financial actions
Deleting important data
Changing system configuration
Controlling external devices
Making irreversible changes
🔐 Permission Model

Zx should eventually support capability-based permissions.

                    Zx
                         │
                  Permission Layer
                         │
       ┌─────────────────┼─────────────────┐
       ▼                 ▼                 ▼
     READ              WRITE             EXECUTE
       │                 │                 │
       ▼                 ▼                 ▼
   Low Risk          Medium Risk        High Risk

The agent should never automatically assume unlimited access.

🎨 User Interface

The Zx interface is designed as a futuristic AI command center.

The interface concept includes:

🌌 Central AI avatar
🧠 AI core visualization
📊 System monitoring
🎙️ Voice status
🧩 Agent controls
💾 Memory controls
🔌 Plugin controls
👁️ Vision interface
📱 Phone control
💬 Conversation interface
⚙️ Settings
📜 Activity console
🔄 Autonomous agent loop

The UI is intended to provide a clear visual representation of what the AI is doing internally.

🧭 Development Roadmap
Phase 1 — SANU06 Foundation
 LLM interaction
 Voice system
 Personality system
 Memory foundation
 Plugin architecture
 Computer-related functionality
Phase 2 — Zx Core
 Agent architecture
 Goal management
 Task planner
 Multi-step execution
 Execution state management
 Self-evaluation
 Error recovery
 Improved memory architecture
Phase 3 — Tool Intelligence
 Computer control
 Phone control
 Web research
 Coding agent
 Debugging agent
 Data analysis
 File intelligence
Phase 4 — Multi-Agent Intelligence
 Agent coordinator
 Specialized agents
 Agent delegation
 Agent communication
 Parallel task execution
 Agent result evaluation
Phase 5 — Learning & Knowledge
 Experience memory
 Feedback system
 Knowledge management
 Adaptive planning
 Successful-solution retrieval
 Failure memory
 Experiment tracking
Phase 6 — Advanced AGI Research
 Long-horizon planning
 General task solving
 Persistent world models
 Advanced reasoning
 Autonomous experimentation
 Improved self-evaluation
 Robust recovery
 General-purpose agent evaluation
🗺️ Long-Term Architecture

The long-term direction can be represented as:

                        Zx
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       Reasoning         Memory            Tools
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                         Planning
                            │
                            ▼
                         Agents
                            │
                            ▼
                         Actions
                            │
                            ▼
                        Evaluation
                            │
                            ▼
                       Correction
                            │
                            ▼
                          Learning
                            │
                            └──────────► Repeat
🧩 Technology Direction

Zx is designed as a modular system rather than a single model.

The architecture can combine:

LLMs
 │
 ├── Reasoning
 ├── Planning
 └── Generation
       │
       ▼
Agent Runtime
       │
       ├── Memory
       ├── Tools
       ├── Plugins
       ├── Computer Control
       ├── Phone Control
       └── External Services
              │
              ▼
        Evaluation Layer
              │
              ▼
        Learning Layer

The exact technologies may evolve as the research progresses.

📁 Project Structure

A simplified conceptual structure:

Zx/
│
├── plugins/
│   ├── calendar_control.py
│   ├── calculator.py
│   ├── chat_takeover.py
│   ├── excel_writer.py
│   ├── gmail_control.py
│   ├── home_assistant.py
│   ├── pomodoro.py
│   ├── printer_control.py
│   ├── shopping_list.py
│   ├── spaced_repetition.py
│   ├── upload_video.py
│   ├── water_reminder.py
│   └── wireless_control.py
│
├── ...
│
└── README.md

The repository will continue to evolve as new components are introduced.

📈 Project Evolution
                     Zx
                       │
                       ▼
                ┌─────────────┐
                │ SANU06  │
                │ AI Assistant│
                └──────┬──────┘
                       │
                       ▼
                ┌─────────────┐
                │ Zx  │
                │ AGI Research│
                └──────┬──────┘
                       │
                       ▼
                ┌─────────────┐
                │ Advanced AGI│
                │   Research  │
                └──────┬──────┘
                       │
                       ▼
                ┌─────────────┐
                │ Future ASI  │
                │   Research  │
                └─────────────┘
🧪 Research Principles

Zx development follows several principles:

1. Build Systematically

New capabilities should integrate into a coherent architecture rather than being random standalone features.

2. Measure Progress

Capabilities should be evaluated through repeatable tasks and benchmarks.

3. Verify Actions

The system should verify whether an action actually achieved its intended result.

4. Learn From Failure

Failed executions should provide useful information for future attempts.

5. Maintain Human Control

Important actions should remain subject to user permissions and approval.

6. Be Realistic

Current AI systems have significant limitations.

Zx is therefore treated as an experimental research platform rather than proof that true AGI has been achieved.

⚠️ Limitations

Zx does not currently represent true Artificial General Intelligence.

Current AI systems can still experience:

Hallucinations
Reasoning failures
Incorrect assumptions
Tool-use errors
Context limitations
Planning failures
Incomplete world knowledge
Poor long-horizon reliability

The purpose of Zx is to research architectures that may improve these capabilities.

🎯 First Major Milestone

The first major Zx milestone is:

Build a reliable autonomous agent loop.

The initial target is:

User Goal
    ↓
Understand
    ↓
Plan
    ↓
Execute
    ↓
Evaluate
    ↓
Correct
    ↓
Complete

The system should successfully complete multi-step tasks while:

Maintaining context
Selecting appropriate tools
Handling errors
Verifying results
Asking for approval when required
Recording useful information

This becomes the foundation for every future Zx capability.

🚀 Future

The ultimate objective is not simply to create a chatbot.

The objective is to explore whether a modular AI system can progressively combine:

Reasoning
+
Planning
+
Memory
+
Learning
+
Tools
+
Computer Use
+
Multi-Agent Collaboration
+
Self-Evaluation
+
Knowledge
+
Experimentation

into a more capable general-purpose AI system.

📊 Current Status
PROJECT
Zx

TYPE
AGI Research Prototype

CURRENT STAGE
Active Development

PREVIOUS VERSION
SANU06

PRIMARY FOCUS
Autonomous General-Purpose AI

CORE LOOP
Goal → Understand → Plan → Execute → Evaluate → Correct → Learn

STATUS
🟡 ACTIVE DEVELOPMENT
👨‍💻 Project

Zx

AI Companion • AGI Research Prototype

LEARN • BUILD • EXPLORE • EVOLVE

⚠️ Disclaimer

Zx is an experimental research and engineering project.

It explores architectures, techniques, and software systems related to increasingly capable general-purpose AI.

It does not claim to have achieved Artificial General Intelligence (AGI), Artificial Superintelligence (ASI), consciousness, or human-level general intelligence.

Any future claims about capability should be supported by measurable evaluations and reproducible results.
