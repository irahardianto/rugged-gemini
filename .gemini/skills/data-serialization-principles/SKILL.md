---
name: data-serialization-principles
description: >-
  Serialization best practices: schema validation, encoding, versioning,
  backward compat, safe deserialization. JSON, YAML, Protobuf, MessagePack.
user-invocable: false
---

## Data Serialization Principles

### Validate at Boundaries
All incoming data validated: API requests, file uploads, MQ messages. Type, format, range, required fields. Fail fast on invalid. Clear errors to client.

### Encoding
Default UTF-8 everywhere. Specify explicitly when reading/writing. Handle errors gracefully.

### Format Selection

| Format | Good For | Limitations |
|---|---|---|
| JSON | APIs, config, web (human-readable, universal) | No binary, larger size |
| Protobuf | Internal microservices, high-throughput (compact, schema evolution) | Not readable, requires schema |
| MessagePack | Internal APIs needing speed over JSON | Less widely supported |
| XML | Enterprise, SOAP, RSS (legacy) | Verbose, XXE attacks |
| YAML | Config, IaC (K8s, CI/CD) (human-friendly) | Complex parsing, security risks |

### Security
- Validate before deserialization (prevent arbitrary code execution)
- Size limits on payloads (prevent OOM)
- Whitelist allowed types/classes
- XML: disable external entities (XXE)
- YAML: disable unsafe constructors
- Python pickle: NEVER deserialize untrusted data

### Related
- Error Handling GEMINI.md § Error Handling Principles
- Security Mandate GEMINI.md § Security Mandate
- Security Principles GEMINI.md § Security Principles
- API Design @.gemini/skills/api-design-principles/SKILL.md
