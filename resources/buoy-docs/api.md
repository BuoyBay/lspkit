---
title: "LSPKit API"
summary: "Public types and operations supplied by LSPKit."
category: "Reference"
order: 20
keywords: ["LSP", "JSON-RPC", "framing", "language-server", "protocol"]
symbols: ["Request", "Notification", "SuccessResponse", "ErrorResponse", "ValidateEnvelope", "EncodeFrame", "LSPFrameDecoder", "LSPFrameDecoder.Feed", "LSPFrameDecoder.Finish", "LSPFrameDecoder.Reset", "LSPRequestTracker", "LSPRequestTracker.Track", "LSPRequestTracker.Complete", "LSPRequestTracker.Forget"]
related: ["index.md", "getting-started.md", "typed-api.md"]
---

# API reference

## Envelope helpers

| Symbol | Contract |
| --- | --- |
| `Request(requestId, methodName, arguments)` | Request with string or signed 32-bit integer JSON ID. |
| `Notification(methodName, arguments)` | Message without an ID. |
| `SuccessResponse(requestId, payload)` | Successful result; Nil payload becomes JSON null. |
| `ErrorResponse(requestId, errorCode, message, details)` | Error with signed 32-bit code; Nil ID becomes null and Nil details omit data. |
| `ValidateEnvelope(message)` | Validate one JSON-RPC 2.0 request, notification, or response; not a batch. |
| `EncodeFrame(message, maxBodyBytes = 8388608)` | Validate and encode ASCII header plus UTF-8 JSON body measured in bytes. |

Responses require exactly one of result/error. A successful response cannot have a null ID. Params cannot appear on a response. Method-specific payload validation belongs to the client.

## Decoder

`LSPFrameDecoder(maxBodyBytes = 8388608, maxHeaderBytes = 8192, maxMessagesPerFeed = 256, maxFeedBytes = 67108864)` bounds body, header, returned messages, and each input feed. `Feed(chunk)` returns completed validated messages in order and retains incomplete bytes. `HasPendingData` reports retained framing state; `IsFailed` reports poisoning. `Finish()` checks EOF and `Reset()` clears pending/failed state.

## Request tracker

`LSPRequestTracker(maxPending = 1024)` exposes `PendingCount`. `Track(request)` rejects duplicates, notifications, and capacity overflow. `Complete(response)` removes the matching ID and returns the method name; unknown/duplicate/null-ID responses reject. `Forget(requestId)` removes local state and reports whether it existed. Failed validation leaves the pending set unchanged.

[Getting started](getting-started.md) · [Package overview](index.md)

[Full typed API: properties, parameters, defaults, and returns](typed-api.md)

A server may still respond after cancellation. Keep the request tracked until its response arrives when possible. After `Forget`, a late response is unknown and `Complete` rejects it without changing other pending requests; the caller must handle that response. Avoid reusing a forgotten ID while its old response can still arrive, because correlation uses only the ID and cannot distinguish the old request from a new one.
