# LSP JSON-RPC framing

Source-only building blocks for an IDE's language-server connection: byte framing,
JSON-RPC envelopes, and bounded request correlation. `Import lspkit` after placing
`lspkit-0.1.0.buipkg` in the consumer project's `packages/` directory.

This package implements the [LSP 3.17 base protocol](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#baseProtocol).
It uses Buoy's built-in `JSONItem`, `Json` and `Bytes`; it has no native library
or package dependency. It does not launch a language server, own a transport,
implement language features or negotiate server capabilities. Feed bytes from
your chosen stdio, socket or pipe transport and write encoded bytes back to it.
Buoy's `Shell` merges stdout and stderr, so its output alone is unsuitable as a
clean LSP stdio transport.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add lspkit --git https://github.com/BuoyBay/lspkit.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## Example

```basic
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

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer lspkit
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
