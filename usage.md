# Usage

## Command-line

To transcode an archetype file `escrow.arl` to `michelson`:

```text
$ archetype escrow.arl
```

To transcode to `whyml`:

```text
$ archetype -t whyml escrow.arl
```

To list available target languages:

```text
$ archetype --list-target
  ligo
  whyml
```

To transcode to `Ligo`:

```text
$ archetype -t ligo escrow.arl
```

To list available commands:

```bash
$ archetype --help
usage : archetype [-t <lang> | -pt | -ast | -mdl | -c | -ir | -lsp <request>] [-r ] <file>

Available options:
  -c compile                     to michelson
  --compile                      Same as -c
  -t <lang>                      Transcode to <lang> language
  --target                       Same as -t
  --list-target                  List available target languages
  -pt                            Generate parse tree
  --parse-tree                   Same as -pt
  -ast                           Generate typed ast
  --typed-ast                    Same as -ast
  -mdl                           Generate model
  --model                        Same as -mdl
  -fp                            Focus property (with whyml target only)
  --focus-property               Same as -fp
  -sci                           Set caller address for initialization
  --set-caller-init              Same as -sci
  -lsp <request>                 Generate language server protocol response to <resquest>
  --list-lsp-request             List available request for lsp
  --service <service>            Generate service response to <service>
  --list-services                List available services
  -m                             Pretty print model tree
  --model                        Same as -m
  -r                             Print raw model tree
  --raw                          Same as -r
  -ry                            Print raw model tree
  --raw-whytree                  Same as -r
  -ir                            Generate intermediate representation
  --intermediate-representation  Same as -ir
  -ri                            Print raw intermediate representation
  --raw-ir                       Same as -ri
  -rm                            Print raw michelson
  --raw-michelson                Same as -rm
  -V <id>                        process specication identifiers
  -v                             Show version number and exit
  --version                      Same as -v
  -help                          Display this list of options
  --help                         Display this list of options
```

## VS code extension

The archetype extension provides:

* syntax highlighting
* [LSP](https://microsoft.github.io/language-server-protocol/) support
* transcoding commands

![](.gitbook/assets/screenshot-2020-06-25-at-14.10.42.png)

The archetype extension provides commands to compile to Michelson via Ligo, and to launch the why3 IDE for verification:

![](.gitbook/assets/capture-de-cran-2020-09-12-a-15.38.13.png)



