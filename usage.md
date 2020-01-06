# Usage

## Command line

To transcode say an archetype file escrow.arl to `ligo`:

```text
$ archetype -t ligo escrow.arl
```

To transcode to `whyml`:

```text
$ archetype -t whyml escrow.arl
```

To list available target languages:

```text
$ archetype --list-target
  ligo
  smartpy
  ocaml
  whyml
  markdown
```

To list available commands:

```bash
$ archetype --help
usage : archetype [-t <lang> | -pt | -ext | -tast | [-as] [-nse] | -lsp <request>] [-r | -json] <file>

Available options:
  -t <lang>           Transcode to <lang> language
  --target            Same as -t
  --list-target       List available target languages
  -pt                 Generate parse tree
  --parse-tree        Same as -pt
  -ext                Process extensions
  --extensions        Same as -ext
  -tast               Generate typed ast
  --typed-ast         Same as -tast
  -sa                 Transform to shallow asset
  --shallow-asset     Same as -sa
  -nse                Remove side effect
  --no-side-effect    Same as -nse
  -lsp <request>      Generate language server protocol response to <resquest>
  --list-lsp-request  List available request for lsp
  -r                  Print raw tree
  --raw               Same as -r
  -json               Print JSON format
  -help               Display this list of options
  --help              Display this list of options

```

{% hint style="warning" %}
 Some features are still under development
{% endhint %}

## VS code extension

The archetype extension provides:

* syntax highlighting
* [LSP](https://microsoft.github.io/language-server-protocol/) support
* transcoding commands

![](.gitbook/assets/screenshot-2019-08-04-at-14.59.59.png)

