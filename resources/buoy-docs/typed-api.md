---
title: "lspkit typed API"
summary: "Source-derived public signatures, properties, parameters, defaults, and return values."
category: "Reference"
order: 21
types: ["LSPFrameDecoder", "LSPRequestTracker"]
dependencies: []
symbols: ["LSPFrameDecoder", "LSPRequestTracker"]
related: ["api.md", "getting-started.md", "index.md"]
---

# lspkit typed API

Generated from the package source declarations. Platform guards are shown where present; inherited members are documented on the base type. See the [API contracts](api.md) for behavior and limitations.

## LSPKit functions

### ValidateEnvelope

`Public Sub ValidateEnvelope(message As JSONItem)`

Validate one LSP JSON-RPC request, notification or response (batch arrays are unsupported).

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `JSONItem` | `ByVal` | Required |

Returns: no value.

### Request

`Public Function Request(requestId As JSONItem, methodName As String, arguments As JSONItem) As JSONItem`

Pass Nil for arguments to omit params; JSON null is not valid LSP params.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requestId` | `JSONItem` | `ByVal` | Required |
| `methodName` | `String` | `ByVal` | Required |
| `arguments` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

### Notification

`Public Function Notification(methodName As String, arguments As JSONItem) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `methodName` | `String` | `ByVal` | Required |
| `arguments` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

### SuccessResponse

`Public Function SuccessResponse(requestId As JSONItem, payload As JSONItem) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requestId` | `JSONItem` | `ByVal` | Required |
| `payload` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

### ErrorResponse

`Public Function ErrorResponse(requestId As JSONItem, errorCode As Integer, message As String, details As JSONItem) As JSONItem`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requestId` | `JSONItem` | `ByVal` | Required |
| `errorCode` | `Integer` | `ByVal` | Required |
| `message` | `String` | `ByVal` | Required |
| `details` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `JSONItem` |

### EncodeFrame

`Public Function EncodeFrame(message As JSONItem, maxBodyBytes As Integer = 8388608) As Bytes`

UTF-8 body length is measured in bytes; the returned buffer owns header and body.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `JSONItem` | `ByVal` | Required |
| `maxBodyBytes` | `Integer` | `ByVal` | `8388608` |

#### Returns

| Type |
| --- |
| `Bytes` |

## LSPFrameDecoder

`Public Class LSPFrameDecoder`

Incremental single-threaded decoder. A protocol failure poisons it until Reset.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `IsFailed` | `Boolean` | Yes | No | — | — |
| `HasPendingData` | `Boolean` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(maxBodyBytes As Integer = 8388608, maxHeaderBytes As Integer = 8192, maxMessagesPerFeed As Integer = 256, maxFeedBytes As Integer = 67108864)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `maxBodyBytes` | `Integer` | `ByVal` | `8388608` |
| `maxHeaderBytes` | `Integer` | `ByVal` | `8192` |
| `maxMessagesPerFeed` | `Integer` | `ByVal` | `256` |
| `maxFeedBytes` | `Integer` | `ByVal` | `67108864` |

Returns: no value.

### Reset

`Public Sub Reset()`

No parameters.

Returns: no value.

### Feed

`Public Function Feed(chunk As Bytes) As Array Of JSONItem`

Returns completed validated messages in wire order; unfinished bytes remain owned by the decoder.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `chunk` | `Bytes` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Array Of JSONItem` |

### Finish

`Public Sub Finish()`

Signal transport EOF; an unfinished header/body is a protocol error.

No parameters.

Returns: no value.

## LSPRequestTracker

`Public Class LSPRequestTracker`

Correlates out-of-order responses without conflating numeric and string IDs.

### Properties

| Name | Type | Read-Only | Shared | Default | Description |
| --- | --- | --- | --- | --- | --- |
| `PendingCount` | `Integer` | Yes | No | — | — |

### Constructor

`Public Sub Constructor(maxPending As Integer = 1024)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `maxPending` | `Integer` | `ByVal` | `1024` |

Returns: no value.

### Track

`Public Sub Track(message As JSONItem)`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `JSONItem` | `ByVal` | Required |

Returns: no value.

### Complete

`Public Function Complete(message As JSONItem) As String`

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `message` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `String` |

### Forget

`Public Function Forget(requestId As JSONItem) As Boolean`

Removes local correlation state only; send $/cancelRequest separately when needed.

#### Parameters

| Name | Type | Passing | Default |
| --- | --- | --- | --- |
| `requestId` | `JSONItem` | `ByVal` | Required |

#### Returns

| Type |
| --- |
| `Boolean` |
