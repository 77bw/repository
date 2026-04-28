# oh-my-opencode.json 配置文件

> Oh My OpenCode 的 Agent 和 Category 配置
> 

---

```
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    },
    "oracle": {
      "model": "codexzh/gpt-5.2-codex"
    },
    "librarian": {
      "model": "antigravity-tools/gemini-3-flash"
    },
    "explore": {
      "model": "antigravity-tools/gemini-3-pro-high"
    },
    "multimodal-looker": {
      "model": "antigravity-tools/gemini-3-pro-high"
    },
    "prometheus": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    },
    "metis": {
      "model": "codexzh/gpt-5.2"
    },
    "momus": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    },
    "atlas": {
      "model": "antigravity-tools/gemini-3-pro-high"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "antigravity-tools/gemini-3-pro-high"
    },
    "ultrabrain": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    },
    "quick": {
      "model": "antigravity-tools/gemini-3-flash"
    },
    "artistry": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    },
    "unspecified-low": {
      "model": "codexzh/gpt-5.2"
    },
    "unspecified-high": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    },
    "writing": {
      "model": "antigravity-tools/claude-opus-4-5-thinking"
    }
  },
  "sisyphus_agent": {
    "disabled": false,
    "planner_enabled": true,
    "replace_plan": true
  }
}
```