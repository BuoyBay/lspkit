---
title: "LSPKit getting started"
summary: "Install LSPKit and try a complete usage example."
category: "Guides"
order: 10
keywords: ["LSP", "JSON-RPC", "framing", "language-server", "protocol"]
related: ["index.md", "api.md", "typed-api.md"]
---

# Getting started

Place `lspkit-0.1.0.buipkg` in `packages/` and import `lspkit`.

## Encode and correlate a response

The example uses an in-memory transport round trip. A real client would write the encoded request to its transport and feed arbitrary incoming byte fragments to the decoder.

```buoy
Import lspkit

Sub Main()
    Dim parameters As JSONItem = JSONItem.NewObject()
    parameters.Value("capabilities") = JSONItem.NewObject()
    Dim request As JSONItem = Request(JSONItem.From(1.0), "initialize", parameters)
    Dim tracker As New LSPRequestTracker()
    tracker.Track(request)
    Dim outgoing As Bytes = EncodeFrame(request)
    Print "Request bytes: " + String.From(outgoing.Count)

    Dim payload As JSONItem = JSONItem.NewObject()
    payload.Value("capabilities") = JSONItem.NewObject()
    Dim decoder As New LSPFrameDecoder()
    For Each response As JSONItem In decoder.Feed(EncodeFrame(SuccessResponse(JSONItem.From(1.0), payload)))
        Print "Completed: " + tracker.Complete(response)
    Next
    decoder.Finish()
End Sub
```

## Finish the stream deliberately

Call `Finish()` at transport EOF to detect an incomplete header/body. Use `Reset()` only when beginning a new stream or at a separately established frame boundary. Do not reset and continue guessing after corruption.

Pass `Nil` arguments to omit params. Non-Nil params must be an object or array; explicit JSON null rejects. Protocol violations use `InvalidArgumentException`; JSON/encoding errors preserve their standard-library exception types. Forgetting a tracked request releases correlation state but sends no cancellation notification.

[Package overview](index.md) · [API reference](api.md)
