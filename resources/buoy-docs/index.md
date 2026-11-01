---
title: "LSPKit"
summary: "Frame LSP JSON-RPC bytes and correlate bounded outstanding requests."
category: "Overview"
order: 0
keywords: ["LSP", "JSON-RPC", "framing", "language-server", "protocol"]
related: ["getting-started.md", "api.md", "typed-api.md"]
---

# LSPKit

LSPKit supplies the base protocol pieces for a language-server connection: Content-Length byte framing, JSON-RPC envelope construction/validation, and request correlation. It uses built-in JSON and byte types, with no native library or package dependency.

It does not start a server, own a transport, negotiate capabilities, implement language features, or validate every method-specific payload. Feed bytes from a host-owned pipe or socket and send encoded bytes through that same transport. Keep stderr separate from LSP stdout; combined shell output is not a clean protocol stream.

## Identity and failures

IDs remain JSON values: numeric `7` and string `"7"` are distinct, and string IDs are case-sensitive. A decoder failure poisons the stream until reset. A failed feed returns no messages from that call, even if valid frames preceded the bad bytes.

Keep decoders and trackers on one thread or synchronize them externally. Backpressure, deadlines, process ownership, and dispatch remain application responsibilities.

## Read next

- [Getting started](getting-started.md)
- [API reference](api.md)
