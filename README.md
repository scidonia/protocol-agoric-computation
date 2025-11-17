
# Protocol for Agoric Computation (PAC)
### RFC: A Standard for Metered Computation, Price Model Disclosure, and Charge Reporting

**Status:** Draft
**Version:** `acp-1`  
**Authors:** _TBD_  
**Intended scope:** Public and private APIs exposing metered computation  
**Last Updated:** 2025-11-17  

# 1. Introduction

This document specifies the **Protocol for Agoric Computation (PAC)**, a standard for discovering and reporting price models for computation. This includes measuring and reporting resource consumption and charges in machine readable, verifiable, deterministic and easily auditable manner. 

This document defines the **Protocol for Agoric Computation (PAC)**, a standard for declaring
price models for computation, measuring resource consumption, and reporting charges in a
machine-verifiable, deterministic, auditable way.

It is transport-agnostic, language-agnostic, and billing-system‑agnostic.

# 2. Terminology

- **Endpoint** — A computational service.
- **Client** — An entity consuming computation.
- **Resource dimension** — A measurable quantity: bytes, tokens, time.
- **Tokenisation model** — Deterministic byte→token mapping.
- **Price model** — Declarative cost structure.
- **Charge report** — Resource usage + computed charges.

# 3. Motivation



# 4. Resource Dimensions

PAC defines three different resource dimensions, each of which can be associated with a given resource as input or output. Some endpoints will have multiple inputs and multiple outputs, all having different pricing models.

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
PAC-Charge-Report: none | summary | full
```

---

# 9. Measurement Requirements

Defines bytes, tokens, time measurement rules. CPU vs wall time distinction required.

---

# 10. JSON Schema — Price Model

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://example.com/schemas/acp-price-model.json",
  "title": "ACP Price Model",
  "type": "object",
  "required": ["acp_version", "model_id", "components"],
  "properties": {
    "acp_version": {
      "type": "string",
      "enum": ["acp-1"]
    },
    "model_id": {
      "type": "string",
      "minLength": 1
    },
    "components": {
      "type": "array",
      "minItems": 1,
      "items": { "$ref": "#/$defs/priceComponent" }
    },
    "fixed_fees": {
      "type": "array",
      "items": { "$ref": "#/$defs/fixedFee" }
    },
    "modifiers": {
      "type": "array",
      "items": { "$ref": "#/$defs/modifier" }
    },
    "terms_uri": {
      "type": "string",
      "format": "uri"
    }
  },
  "$defs": {
    "resource": {
      "type": "object",
      "required": ["kind"],
      "properties": {
        "kind": {
          "type": "string",
          "enum": ["bytes", "tokens", "time"]
        },
        "direction": {
          "type": "string",
          "enum": ["in", "out", "bidirectional"]
        },
        "token_model_id": {
          "type": "string"
        },
        "subtype": {
          "type": "string",
          "enum": ["cpu", "wall"]
        }
      },
      "allOf": [
        {
          "if": {
            "properties": { "kind": { "const": "tokens" } }
          },
          "then": {
            "required": ["token_model_id", "direction"]
          }
        },
        {
          "if": {
            "properties": { "kind": { "const": "bytes" } }
          },
          "then": {
            "required": ["direction"]
          }
        },
        {
          "if": {
            "properties": { "kind": { "const": "time" } }
          },
          "then": {
            "required": ["subtype"]
          }
        }
      ]
    },
    "rate": {
      "type": "object",
      "required": ["amount", "currency", "per"],
      "properties": {
        "amount": {
          "type": "string",
          "pattern": "^-?[0-9]+(\\.[0-9]+)?$"
        },
        "currency": {
          "type": "string",
          "minLength": 1
        },
        "per": {
          "type": "object",
          "required": ["quantity"],
          "properties": {
            "quantity": {
              "type": "integer",
              "minimum": 1
            }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },
    "moneyAmount": {
      "type": "object",
      "required": ["value", "currency"],
      "properties": {
        "value": {
          "type": "string",
          "pattern": "^-?[0-9]+(\\.[0-9]+)?$"
        },
        "currency": {
          "type": "string",
          "minLength": 1
        }
      },
      "additionalProperties": false
    },
    "priceComponent": {
      "type": "object",
      "required": ["id", "resource", "rate", "minimum_granularity", "rounding"],
      "properties": {
        "id": {
          "type": "string",
          "minLength": 1
        },
        "description": {
          "type": "string"
        },
        "resource": {
          "$ref": "#/$defs/resource"
        },
        "rate": {
          "$ref": "#/$defs/rate"
        },
        "unit_label": {
          "type": "string"
        },
        "minimum_granularity": {
          "type": "integer",
          "minimum": 1
        },
        "rounding": {
          "type": "string",
          "enum": ["ceil", "floor", "round_half_up"]
        }
      },
      "additionalProperties": false
    },
    "fixedFee": {
      "type": "object",
      "required": ["id", "amount"],
      "properties": {
        "id": {
          "type": "string",
          "minLength": 1
        },
        "description": {
          "type": "string"
        },
        "amount": {
          "$ref": "#/$defs/moneyAmount"
        }
      },
      "additionalProperties": false
    },
    "modifier": {
      "type": "object",
      "required": ["id", "type"],
      "properties": {
        "id": {
          "type": "string",
          "minLength": 1
        },
        "description": {
          "type": "string"
        },
        "type": {
          "type": "string",
          "enum": ["multiplier", "surcharge", "discount"]
        },
        "range": {
          "type": "object",
          "properties": {
            "min": { "type": "number" },
            "max": { "type": "number" }
          },
          "additionalProperties": false
        },
        "deterministic_rule_uri": {
          "type": "string",
          "format": "uri"
        }
      },
      "additionalProperties": false
    }
  }
}
```

---

# 11. JSON Schema — Charge Report

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://example.com/schemas/acp-charge-report.json",
  "title": "ACP Charge Report",
  "type": "object",
  "required": ["acp_version", "model_id", "request_id", "timestamp", "measures", "charges", "total"],
  "properties": {
    "acp_version": {
      "type": "string",
      "enum": ["acp-1"]
    },
    "model_id": {
      "type": "string",
      "minLength": 1
    },
    "request_id": {
      "type": "string",
      "minLength": 1
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "measures": {
      "type": "array",
      "minItems": 1,
      "items": { "$ref": "#/$defs/measure" }
    },
    "charges": {
      "type": "array",
      "minItems": 1,
      "items": { "$ref": "#/$defs/chargeEntry" }
    },
    "modifiers_applied": {
      "type": "array",
      "items": { "$ref": "#/$defs/modifierEntry" }
    },
    "total": {
      "type": "object",
      "required": ["amount"],
      "properties": {
        "amount": { "$ref": "#/$defs/moneyAmount" }
      },
      "additionalProperties": false
    },
    "audit": {
      "type": "object",
      "properties": {
        "input_token_checksum": { "type": "string" },
        "measurement_method": {
          "type": "string"
        }
      },
      "additionalProperties": true
    }
  },
  "$defs": {
    "resource": {
      "type": "object",
      "required": ["kind"],
      "properties": {
        "kind": {
          "type": "string",
          "enum": ["bytes", "tokens", "time"]
        },
        "direction": {
          "type": "string",
          "enum": ["in", "out", "bidirectional"]
        },
        "token_model_id": {
          "type": "string"
        },
        "subtype": {
          "type": "string",
          "enum": ["cpu", "wall"]
        }
      },
      "allOf": [
        {
          "if": {
            "properties": { "kind": { "const": "tokens" } }
          },
          "then": {
            "required": ["token_model_id", "direction"]
          }
        },
        {
          "if": {
            "properties": { "kind": { "const": "bytes" } }
          },
          "then": {
            "required": ["direction"]
          }
        },
        {
          "if": {
            "properties": { "kind": { "const": "time" } }
          },
          "then": {
            "required": ["subtype"]
          }
        }
      ]
    },
    "moneyAmount": {
      "type": "object",
      "required": ["value", "currency"],
      "properties": {
        "value": {
          "type": "string",
          "pattern": "^-?[0-9]+(\\.[0-9]+)?$"
        },
        "currency": {
          "type": "string",
          "minLength": 1
        }
      },
      "additionalProperties": false
    },
    "rate": {
      "type": "object",
      "required": ["amount", "currency", "per"],
      "properties": {
        "amount": {
          "type": "string",
          "pattern": "^-?[0-9]+(\\.[0-9]+)?$"
        },
        "currency": {
          "type": "string",
          "minLength": 1
        },
        "per": {
          "type": "object",
          "required": ["quantity"],
          "properties": {
            "quantity": {
              "type": "integer",
              "minimum": 1
            }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },
    "measure": {
      "type": "object",
      "required": ["resource", "quantity"],
      "properties": {
        "resource": { "$ref": "#/$defs/resource" },
        "quantity": {
          "type": "integer",
          "minimum": 0
        }
      },
      "additionalProperties": false
    },
    "chargeEntry": {
      "type": "object",
      "required": ["component_id", "amount"],
      "properties": {
        "component_id": {
          "type": "string",
          "minLength": 1
        },
        "quantity": {
          "type": "integer",
          "minimum": 0
        },
        "rate": {
          "$ref": "#/$defs/rate"
        },
        "amount": {
          "$ref": "#/$defs/moneyAmount"
        }
      },
      "additionalProperties": false
    },
    "modifierEntry": {
      "type": "object",
      "required": ["modifier_id", "value", "amount_delta"],
      "properties": {
        "modifier_id": {
          "type": "string",
          "minLength": 1
        },
        "value": {
          "type": "number"
        },
        "amount_delta": {
          "$ref": "#/$defs/moneyAmount"
        }
      },
      "additionalProperties": false
    }
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

PAC defines a deterministic, auditable economic protocol for computation.

