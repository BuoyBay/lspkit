# LSPKit · 0.1.0

Pure Buoy building blocks for an IDE's language-server connection: byte framing,
JSON-RPC envelopes, and bounded request correlation. `Import lspkit` after placing
`lspkit-0.1.0.buipkg` in the consumer project's `packages/` directory.

This package implements the [LSP 3.17 base protocol](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#baseProtocol).
It uses Buoy's built-in `JSONItem`, `Json` and `Bytes`; it has no native library
or package dependency. It does not launch a language server, own a transport,
implement language features or negotiate server capabilities. Feed bytes from
your chosen stdio, socket or pipe transport and write encoded bytes back to it.
Buoy's `Shell` merges stdout and stderr, so its output alone is unsuitable as a
clean LSP stdio transport.

## API

| API | Behavior |
| --- | --- |
| `Request(id, methodName, arguments)` | Create a request; `id` is a string or signed 32-bit integer `JSONItem`. |
| `Notification(methodName, arguments)` | Create a notification without an ID. |
| `SuccessResponse(id, payload)` | Create a successful response; `Nil` payload becomes JSON null. |
| `ErrorResponse(id, code, message, details)` | Create an error; `Nil` ID becomes JSON null, `Nil` details omit data. |
| `ValidateEnvelope(message)` | Validate one LSP JSON-RPC 2.0 envelope. |
| `EncodeFrame(message, maxBodyBytes = 8388608)` | Validate and encode an ASCII header followed by UTF-8 JSON bytes. |
| `LSPFrameDecoder.Feed(chunk)` | Return all completed validated messages in wire order; retain unfinished bytes. |
| `LSPFrameDecoder.Finish()` | Check EOF: reject any incomplete header or body. |
| `LSPFrameDecoder.Reset()` | Discard pending bytes and clear failure state for a new stream. |
| `LSPRequestTracker.Track(request)` | Register a request ID and method. |
| `LSPRequestTracker.Complete(response)` | Remove the matching request and return its method name. |
| `LSPRequestTracker.Forget(id)` | Remove local correlation state, returning whether it existed. |

Pass `Nil` arguments to omit `params`. Non-Nil arguments must be a JSON object
or array; explicit JSON null is rejected. IDs are `JSONItem` values so a numeric
ID and a string ID such as `7` and `"7"` remain distinct. String IDs are case
sensitive. Error response IDs may be JSON null for parse/invalid-request errors;
those responses cannot complete a tracked request. Responses must contain exactly
one of `result` and `error`. Error codes use the LSP signed 32-bit integer range.

`LSPRequestTracker(maxPending = 1024)` rejects duplicate IDs, excess pending
requests, notifications, unknown responses and duplicate responses. Validation
failure leaves pending requests unchanged. `PendingCount` reports its size.
Forgetting a request does not send cancellation; send a separate `$/cancelRequest`
notification if the application needs that protocol action.

A server may still respond after cancellation. Keep the request tracked until its response arrives when possible. After `Forget`, a late response is unknown and `Complete` rejects it without changing other pending requests; the caller must handle that response. Avoid reusing a forgotten ID while its old response can still arrive, because correlation uses only the ID and cannot distinguish the old request from a new one.

See the [runnable example](../examples/lspkit_demo.bui) for constructing an
initialize request, encoding it, and correlating a simulated response.

## Incremental decoding and bounds

`LSPFrameDecoder(maxBodyBytes = 8388608, maxHeaderBytes = 8192,
maxMessagesPerFeed = 256, maxFeedBytes = 67108864)` handles splits anywhere in a
header, its delimiter or a UTF-8 sequence, as well as multiple frames in one
chunk. Header limits include the terminating CRLF CRLF. Body lengths count UTF-8
bytes. `HasPendingData` reports incomplete input; `IsFailed` reports a failed
decoder. An empty feed is allowed.

The decoder requires one decimal `Content-Length`. Header names are case
insensitive ASCII HTTP tokens. Duplicate names, bare CR/LF, invalid control
bytes, unsupported charset/content type, malformed lengths, excessive input,
invalid UTF-8, JSON parsing errors and invalid envelopes fail the feed. Optional
`Content-Type` accepts `application/vscode-jsonrpc`, with a `charset`
of `utf-8` or the legacy `utf8`, optionally enclosed in double quotes; other
parameters are rejected. Other unique headers are ignored. This is a deliberately strict header subset, without
folding or additional MIME parameters. Batch arrays are unsupported. Nesting is capped at 128 levels before calling
the native JSON parser. Actual JSON `\u0000` escapes reject before parsing because
Buoy 0.72 silently drops decoded NUL and could alias IDs or method names; literal
escaped backslashes followed by `u0000` remain valid.

After any feed error the decoder is poisoned, and later feeds fail until reset.
If a feed contains valid messages followed by invalid data, that call throws and
returns no messages. Treat the transport as failed; reset only when beginning a
new stream or at a separately established frame boundary. `Finish()` detects a
truncated message instead of silently dropping it. JSON parsing and encoding
errors preserve the stdlib exception type; protocol and envelope violations use
`InvalidArgumentException`.

Objects are confined to one thread unless access is externally synchronized.
The application remains responsible for transport backpressure, server lifecycle,
timeouts, method-specific schemas and response dispatch. This release does not
validate every LSP method payload or provide a full language-server client.

## Verification


Maintainers run the private validation suite before release. See the repository README for public build instructions.

