# OpenClaw Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         USER INTERACTION LAYER                           │
│                                                                          │
│  WhatsApp  Telegram  Discord  Signal  Slack  iMessage  Matrix  Teams... │
│     └─────────┴────────┴───────┴───────┴──────┴────────┴───────┴────┘   │
│                              ▲                                           │
│                              │ Messages                                  │
│                              ▼                                           │
└──────────────────────────────────────────────────────────────────────────┘
                               │
                               │
┌──────────────────────────────▼───────────────────────────────────────────┐
│                        CHANNEL PLUGIN LAYER                              │
│                                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│  │ Channel    │  │ Channel    │  │ Channel    │  │ Channel    │       │
│  │ Plugin 1   │  │ Plugin 2   │  │ Plugin 3   │  │ Plugin N   │       │
│  │            │  │            │  │            │  │            │       │
│  │ • Inbound  │  │ • Inbound  │  │ • Inbound  │  │ • Inbound  │       │
│  │ • Outbound │  │ • Outbound │  │ • Outbound │  │ • Outbound │       │
│  │ • Auth     │  │ • Auth     │  │ • Auth     │  │ • Auth     │       │
│  │ • Status   │  │ • Status   │  │ • Status   │  │ • Status   │       │
│  │ • Pairing  │  │ • Pairing  │  │ • Pairing  │  │ • Pairing  │       │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘       │
│         │               │               │               │               │
└─────────┼───────────────┼───────────────┼───────────────┼───────────────┘
          │               │               │               │
          └───────────────┴───────────────┴───────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     GATEWAY SERVER (Control Plane)                       │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                    WebSocket/HTTP Server                      │      │
│  │  • /v1/chat/completions (OpenAI compatible)                  │      │
│  │  • /v1/responses (OpenResponses API)                         │      │
│  │  • WebSocket (Real-time bidirectional)                       │      │
│  │  • Health endpoints                                           │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                          │                                               │
│  ┌─────────────────────┬─┴────────────────────┬──────────────────────┐ │
│  │                     │                      │                      │ │
│  ▼                     ▼                      ▼                      ▼ │
│ ┌─────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│ │Channel  │  │   Agent      │  │   Session    │  │   Plugin     │   │
│ │Manager  │  │   Event      │  │   Manager    │  │   Registry   │   │
│ │         │  │   Handler    │  │              │  │              │   │
│ │• Start  │  │              │  │• History     │  │• Tools       │   │
│ │• Stop   │  │• Route msg   │  │• State       │  │• Hooks       │   │
│ │• Health │  │• Call agent  │  │• Persist     │  │• Channels    │   │
│ └─────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│                     │                                                   │
└─────────────────────┼───────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          AGENT EXECUTION LAYER                           │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │              Pi Agent Framework Integration                   │      │
│  │  (@mariozechner/pi-agent-core, pi-ai, pi-coding-agent)       │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                          │                                               │
│  ┌─────────────────────┬─┴────────────────────┬──────────────────────┐ │
│  │                     │                      │                      │ │
│  ▼                     ▼                      ▼                      ▼ │
│ ┌─────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│ │Tool     │  │   LLM        │  │   Memory     │  │   Sandbox    │   │
│ │Execute  │  │   Provider   │  │   Search     │  │   (Docker)   │   │
│ │         │  │              │  │              │  │              │   │
│ │• bash   │  │• Anthropic   │  │• QMD         │  │• Isolated    │   │
│ │• file   │  │• OpenAI      │  │• LanceDB     │  │• Safe exec   │   │
│ │• message│  │• Bedrock     │  │• Milvus      │  │• Approval    │   │
│ │• custom │  │• Custom      │  │• Vector      │  │  gates       │   │
│ └─────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│       │                                                                 │
└───────┼─────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          HOOK SYSTEM (Cross-Cutting)                     │
│                                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│  │llm_input   │  │llm_output  │  │message_send│  │session_    │       │
│  │            │  │            │  │            │  │reset       │       │
│  │Before LLM  │  │After LLM   │  │Before send │  │On reset    │       │
│  │call        │  │response    │  │            │  │            │       │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘       │
│         │               │               │               │               │
│         └───────────────┴───────────────┴───────────────┘               │
│                          │                                               │
│                          ▼                                               │
│                   Plugin Extensions                                      │
│                   • Custom logging                                       │
│                   • Analytics                                            │
│                   • Custom auth                                          │
│                   • Integrations                                         │
└─────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────┐
│                        CLI & CONFIGURATION LAYER                         │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │                    CLI Commands (Commander.js)                │      │
│  │                                                               │      │
│  │  openclaw gateway     - Start gateway server                 │      │
│  │  openclaw onboard     - Wizard-based setup                   │      │
│  │  openclaw channels    - Manage channels                      │      │
│  │  openclaw agent       - Interact with agent                  │      │
│  │  openclaw config      - Configuration management             │      │
│  │  openclaw doctor      - Diagnostics & repair                 │      │
│  │  openclaw tui         - Terminal UI                          │      │
│  │  openclaw plugin      - Plugin management                    │      │
│  └──────────────────────────────────────────────────────────────┘      │
│                          │                                               │
│  ┌──────────────────────▼──────────────────────────────────────┐      │
│  │                  Configuration (YAML)                         │      │
│  │  • ~/.openclaw/config.yml                                    │      │
│  │  • Environment variable substitution                         │      │
│  │  • Schema validation                                          │      │
│  │  • Hot reload support                                         │      │
│  └──────────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────┐
│                         PLATFORM LAYER                                   │
│                                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│  │  macOS App │  │  iOS App   │  │ Android App│  │   Linux    │       │
│  │  (SwiftUI) │  │  (SwiftUI) │  │  (Kotlin)  │  │  (Systemd) │       │
│  │            │  │            │  │            │  │            │       │
│  │• Menu bar  │  │• Talk Mode │  │• Native UI │  │• Service   │       │
│  │• Sparkle   │  │• Keychain  │  │• Push      │  │• Daemon    │       │
│  │• LaunchD   │  │• Siri      │  │• FCM       │  │• Docker    │       │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘       │
│         │               │               │               │               │
│         └───────────────┴───────────────┴───────────────┘               │
│                          │                                               │
│                          ▼                                               │
│                  Gateway Server (localhost:18789)                        │
└─────────────────────────────────────────────────────────────────────────┘


FLOW EXAMPLE: User sends WhatsApp message → Assistant responds
═══════════════════════════════════════════════════════════════════════════

1. User sends "What's the weather?" via WhatsApp
   │
   ▼
2. WhatsApp Channel Plugin receives message (via Baileys web automation)
   │
   ▼
3. Channel converts to OpenClaw message format
   │
   ▼
4. Gateway's Agent Event Handler receives message
   │
   ▼
5. Session Manager retrieves/creates conversation state
   │
   ▼
6. Agent executes with Pi framework
   │
   ├─▶ Loads conversation history from memory
   ├─▶ Calls LLM (Anthropic Claude) with prompt
   ├─▶ LLM may call tools (e.g., weather API)
   ├─▶ Tool execution (with approval if needed)
   └─▶ LLM generates final response
   │
   ▼
7. Response flows back through Agent Event Handler
   │
   ▼
8. Auto-reply system formats for WhatsApp
   │
   ▼
9. WhatsApp Channel Plugin sends message (via Baileys)
   │
   ▼
10. User receives "Currently 72°F and sunny in San Francisco"


KEY ARCHITECTURAL PRINCIPLES
═════════════════════════════════════════════════════════════════════════

1. **Plugin-First**: Even built-in channels are plugins
2. **Adapter Pattern**: Consistent interface for all channels
3. **Gateway as Control Plane**: Product is the assistant, not the server
4. **Extensibility**: Hooks + Plugin SDK for third-party extensions
5. **Multi-Platform**: CLI, Web, Mobile (iOS/Android), Desktop (macOS)
6. **Self-Hosted**: Privacy-first, runs on your infrastructure
7. **Channel-Agnostic**: Same AI experience across all platforms
8. **Security**: Sandboxing, approval gates, secret management
```
