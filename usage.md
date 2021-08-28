# Usage

## Command-line

To transcode an archetype file `escrow.arl` to `michelson`:

```text
$ completium-cli generate michelson escrow.arl
```

To transcode to `whyml`:

```text
$ completium-cli generate whyml escrow.arl
```

To transcode to `javascript`

```text
$ completium-cli generate javascript escrow.arl
```

The generated javascript provides the Michelson \(Json\) version of the contract code and storage.

## VS code extension

The archetype extension provides:

* syntax highlighting
* [LSP](https://microsoft.github.io/language-server-protocol/) support
* transcoding commands

![](.gitbook/assets/screenshot-2020-06-25-at-14.10.42.png)

The archetype extension provides commands to compile to Michelson, and to launch the why3 IDE for verification:

![](.gitbook/assets/vscode_archetype_commands.png)



