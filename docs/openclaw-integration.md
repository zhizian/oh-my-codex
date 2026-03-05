# OpenClaw / Generic Notification Gateway Integration Guide

This guide covers two supported setup paths:

1. **Explicit OpenClaw schema** (`notifications.openclaw`) — runtime-native shape
2. **Generic aliases** (`custom_webhook_command`, `custom_cli_command`) — flexible setup for OpenClaw or other services

## Activation gates

```bash
# Required for OpenClaw dispatch pipeline
export OMX_OPENCLAW=1

# Required in addition for command gateways
export OMX_OPENCLAW_COMMAND=1
```

## Canonical precedence contract

When both explicit OpenClaw config and generic aliases are present:

1. `notifications.openclaw` wins
2. `custom_webhook_command` / `custom_cli_command` are ignored
3. OMX emits a warning for clarity

This keeps behavior deterministic and backward compatible.

## Option A: Explicit `notifications.openclaw` (legacy/runtime shape)

```json
{
  "notifications": {
    "enabled": true,
    "openclaw": {
      "enabled": true,
      "gateways": {
        "local": {
          "type": "http",
          "url": "http://127.0.0.1:18789/hooks/agent",
          "headers": {
            "Authorization": "Bearer YOUR_HOOKS_TOKEN"
          }
        }
      },
      "hooks": {
        "session-end": {
          "enabled": true,
          "gateway": "local",
          "instruction": "OMX task completed for {{projectPath}}"
        },
        "ask-user-question": {
          "enabled": true,
          "gateway": "local",
          "instruction": "OMX needs input: {{question}}"
        }
      }
    }
  }
}
```

## Option B: Generic aliases (`custom_webhook_command` / `custom_cli_command`)

```json
{
  "notifications": {
    "enabled": true,
    "custom_webhook_command": {
      "enabled": true,
      "url": "http://127.0.0.1:18789/hooks/agent",
      "method": "POST",
      "headers": {
        "Authorization": "Bearer YOUR_HOOKS_TOKEN"
      },
      "events": ["session-end", "ask-user-question"],
      "instruction": "OMX event {{event}} for {{projectPath}}"
    },
    "custom_cli_command": {
      "enabled": true,
      "command": "~/.local/bin/my-notifier --event {{event}} --text {{instruction}}",
      "events": ["session-end"],
      "instruction": "OMX event {{event}} for {{projectPath}}"
    }
  }
}
```

These aliases are normalized by OMX into internal OpenClaw gateway mappings.

## Verification (required)

### A) Wake smoke test (`/hooks/wake`)

```bash
curl -sS -X POST http://127.0.0.1:18789/hooks/wake \
  -H "Authorization: Bearer YOUR_HOOKS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"text":"OMX wake smoke test","mode":"now"}'
```

Expected pass signal: JSON includes `"ok":true`.

### B) Delivery verification (`/hooks/agent`)

```bash
curl -sS -o /tmp/omx-openclaw-agent-check.json -w "HTTP %{http_code}\n" \
  -X POST http://127.0.0.1:18789/hooks/agent \
  -H "Authorization: Bearer YOUR_HOOKS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"instruction":"OMX delivery verification","event":"session-end","sessionId":"manual-check"}'
```

Expected pass signal: HTTP 2xx + accepted response body.

## Preflight checks

```bash
# token present
test -n "$YOUR_HOOKS_TOKEN" && echo "token ok" || echo "token missing"

# reachability
curl -sS -o /dev/null -w "HTTP %{http_code}\n" http://127.0.0.1:18789 || echo "gateway unreachable"

# gate checks
test "$OMX_OPENCLAW" = "1" && echo "OMX_OPENCLAW=1" || echo "missing OMX_OPENCLAW=1"
test "$OMX_OPENCLAW_COMMAND" = "1" && echo "OMX_OPENCLAW_COMMAND=1" || echo "missing OMX_OPENCLAW_COMMAND=1"
```

## Pass/Fail Diagnostics

- **401/403**: invalid/missing bearer token.
- **404**: wrong path; verify `/hooks/agent` and `/hooks/wake`.
- **5xx**: gateway runtime issue; inspect logs.
- **Timeout/connection refused**: host/port/firewall issue.
- **Command gateway disabled**: set both `OMX_OPENCLAW=1` and `OMX_OPENCLAW_COMMAND=1`.
