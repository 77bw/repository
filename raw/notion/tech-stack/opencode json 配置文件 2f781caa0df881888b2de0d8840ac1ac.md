# opencode.json 配置文件

> ⚠️ 注意：此配置使用 Antigravity Tools 反代格式，不能直接用于其他 API 服务！
> 

> 不要直接复制 Claude Desktop、Codex CLI 等工具的配置，格式不兼容。
> 

---

```
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "codexzh": {
      "npm": "@ai-sdk/openai",
      "name": "codexzh",
      "options": {
        "baseURL": "https://api.codexzh.com/v1",
        "apiKey": "sk-fmciWMk415PomdlI3ZkjZ5ixtNbPU4sHBLJw5CjOGHbsYXDI",
        "options": {
          "reasoningEffort": "high",
          "textVerbosity": "low",
          "reasoningSummary": "auto"
        }
      },
      "models": {
        "gpt-5.2": {
          "name": "gpt-5.2"
        },
        "gpt-5.2-codex": {
          "name": "gpt-5.2-codex"
        }
      }
    },
    "antigravity-tools": {
      "npm":"@ai-sdk/anthropic",
      "options": {
        "baseURL": "http://127.0.0.1:8045/v1",
        "apiKey": "sk-01bba4967d274eb080b7ffba0468f7f2"
      },
      "models": {
        "gemini-3-flash": {
          "name": "Gemini 3 Flash (Antigravity)",
          "attachment": true,
          "limit": {
            "context": 1048576,
            "output": 65535
          },
          "modalities": {
            "input": ["text", "image", "pdf"],
            "output": ["text"]
          }
        },
        "gemini-3-pro-high": {
          "name": "Gemini 3 Pro High (Antigravity)",
          "attachment": true,
          "limit": {
            "context": 1048576,
            "output": 65535
          },
          "modalities": {
            "input": ["text", "image", "pdf"],
            "output": ["text"]
          }
        },
        "gemini-3-pro-image": {
          "name": "Gemini 3 Pro Image (Antigravity)",
          "attachment": true,
          "limit": {
            "context": 1048576,
            "output": 65535
          },
          "modalities": {
            "input": ["image"],
            "output": ["image"]
          }
        },
        "claude-sonnet-4-5": {
          "name": "Claude Sonnet 4.5 (Antigravity)",
          "attachment": true,
          "limit": {
            "context": 1048576,
            "output": 65535
          },
          "modalities": {
            "input": ["text", "image", "pdf"],
            "output": ["text"]
          }
        },
        "claude-sonnet-4-5-thinking": {
          "name": "Claude Sonnet 4.5 Think (Antigravity)",
          "attachment": true,
          "limit": {
            "context": 1048576,
            "output": 65535
          },
          "modalities": {
            "input": ["text", "image", "pdf"],
            "output": ["text"]
          }
        },
        "claude-opus-4-5-thinking": {
          "name": "Claude Opus 4.5 Think (Antigravity)",
          "attachment": true,
          "limit": {
            "context": 1048576,
            "output": 65535
          },
          "modalities": {
            "input": ["text", "image", "pdf"],
            "output": ["text"]
          }
        }
      }
    }
  },
  "mcp": {
    "playwright": {
      "type": "local",
      "command": ["npx", "-y", "@playwright/mcp"],
      "enabled": true
    },
    "notion": {
      "type": "local",
      "command": ["npx", "-y", "mcp-remote", "https://mcp.notion.com/mcp"],
      "enabled": true
    }
  },
  "plugin": ["oh-my-opencode"]
}
```