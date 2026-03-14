# I/O Envelope

Every application unit communicates through a standard envelope — the boundary between the application (markup + LLM) and the platform (infrastructure + management).

## Envelope Structure

```
INBOUND:
┌─────────────────────────────────────────────┐
│  Platform Headers                           │
│    request_id, auth_context,                │
│    rate_limit_state, timestamp              │
│                                             │
│  Application Input                          │
│    payload (webhook, API call, event)       │
│                                             │
│  Markup Reference                           │
│    app_id, version, compiled_prompt         │
│                                             │
│  Persistence Context                        │
│    preloaded_state (relevant stored data)   │
└─────────────────────────────────────────────┘

        ↓ LLM "compiles" and executes ↓

OUTBOUND:
┌─────────────────────────────────────────────┐
│  Platform Headers                           │
│    request_id, processing_time_ms,          │
│    token_usage, trace_id, intent_chain      │
│                                             │
│  Application Output                         │
│    response (the result)                    │
│                                             │
│  Side Effects                               │
│    emit: [{ target, payload, method }]      │
│    delegate: [{ target_app, input }]        │
│                                             │
│  Persistence Mutations                      │
│    remember / update / forget declarations  │
│                                             │
│  Management Signals                         │
│    alerts, metrics                          │
└─────────────────────────────────────────────┘
```

The LLM declares what should happen. The platform executes it. Side effects are batched, validated, rate-limited, and audited by the platform.
