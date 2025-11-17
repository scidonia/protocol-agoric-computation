
# Protocol for Agoric Computation (ACP)
### RFC: A Standard for Metered Computation, Price Model Disclosure, and Charge Reporting

**Status:** Draft  
**Version:** `acp-1`  
**Authors:** _TBD_  
**Intended scope:** Public and private APIs exposing metered computation  
**Last Updated:** 2025-11-17  

---

# 1. Introduction

This document defines the **Protocol for Agoric Computation (ACP)**, a standard for declaring
price models for computation, measuring resource consumption, and reporting charges in a
machine-verifiable, deterministic, auditable way.

It is transport-agnostic, language-agnostic, and billing-system‑agnostic.

---

# 2. Terminology

- **Endpoint** — A computational service.
- **Client** — An entity consuming computation.
- **Resource dimension** — A measurable quantity: bytes, tokens, time.
- **Tokenisation model** — Deterministic byte→token mapping.
- **Price model** — Declarative cost structure.
- **Charge report** — Resource usage + computed charges.

---

# 3. Resource Dimensions

ACP defines three:

## 3.1 Bytes
Unit: byte; direction: in/out/bidirectional.

## 3.2 Tokens
Unit: token; requires token_model_id.

## 3.3 Time
Unit: ms; subtype cpu/wall.

---

# 4. Tokenisation Model

Must be deterministic, open, machine-readable.

Example descriptor:

```json
{
  "id": "urn:token-model:example:gpt-bpe-1",
  "name": "Example GPT-BPE v1",
  "version": "1.0.0",
  "spec_uri": "https://example.com/token-models/gpt-bpe-1/spec"
}
```

---

# 5. Pricing Model

Uses structural rate objects:

```json
"rate": {
  "amount": "0.0000020",
  "currency": "ISO-4217:EUR",
  "per": { "quantity": 1 }
}
```

---

# 6. Full Example Pricing Model

```json
{
  "acp_version": "acp-1",
  "model_id": "urn:price-model:example:basic-v1",
  "components": [
    {
      "id": "input-bytes",
      "resource": { "kind": "bytes", "direction": "in" },
      "rate": { "amount": "0.00000001", "currency": "ISO-4217:EUR", "per": { "quantity": 1 } },
      "minimum_granularity": 1,
      "rounding": "ceil"
    }
  ]
}
```

---

# 7. Charge Reporting

Example report:

```json
{
  "acp_version": "acp-1",
  "model_id": "urn:price-model:example:basic-v1",
  "request_id": "xxx",
  "timestamp": "2025-11-17T12:34:56Z",
  "measures": [
    { "resource": { "kind": "bytes", "direction": "in" }, "quantity": 1024 }
  ],
  "charges": [
    {
      "component_id": "input-bytes",
      "quantity": 1024,
      "rate": { "amount": "0.00000001", "currency": "ISO-4217:EUR", "per": { "quantity": 1 } },
      "amount": { "value": "0.00001024", "currency": "ISO-4217:EUR" }
    }
  ],
  "total": { "amount": { "value": "0.00001024", "currency": "ISO-4217:EUR" } }
}
```

---

# 8. HTTP Binding

Discovery:

```
GET /.well-known/acp-price-model
```

Charge report request:

```
ACP-Charge-Report: none | summary | full
```

---

# 9. Measurement Requirements

Defines bytes, tokens, time measurement rules. CPU vs wall time distinction required.

---

# 10. JSON Schema — Price Model

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ACP Price Model",
  "type": "object",
  "required": ["acp_version", "model_id", "components"],
  "properties": {
    "acp_version": { "type": "string" },
    "model_id": { "type": "string" },
    "components": { "type": "array" }
  }
}
```

---

# 11. JSON Schema — Charge Report

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ACP Charge Report",
  "type": "object",
  "required": ["acp_version", "model_id", "request_id", "timestamp", "measures", "total"],
  "properties": {
    "acp_version": { "type": "string" },
    "model_id": { "type": "string" },
    "request_id": { "type": "string" }
  }
}
```

---

# 12. Security Considerations

Measurement integrity, replay protection, privacy considerations.

---

# 13. Versioning

Breaking changes require new acp_version.

---

# 14. Conclusion

ACP defines a deterministic, auditable economic protocol for computation.

