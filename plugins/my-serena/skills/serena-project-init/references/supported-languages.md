# Supported Languages

Complete list of languages supported by Serena's language servers.

## Language List

| Language | Notes |
|----------|-------|
| `al` | AL Language (Business Central) |
| `bash` | Bash/Shell scripts |
| `clojure` | Clojure |
| `cpp` | C++ (also use for C projects) |
| `csharp` | C# (default server) |
| `csharp_omnisharp` | C# (OmniSharp alternative) |
| `dart` | Dart |
| `elixir` | Elixir |
| `elm` | Elm |
| `erlang` | Erlang |
| `fortran` | Fortran |
| `fsharp` | F# |
| `go` | Go |
| `groovy` | Groovy |
| `haskell` | Haskell |
| `hlsl` | HLSL (shaders) |
| `java` | Java |
| `julia` | Julia |
| `kotlin` | Kotlin |
| `lua` | Lua |
| `markdown` | Markdown files |
| `matlab` | MATLAB |
| `nix` | Nix |
| `ocaml` | OCaml |
| `pascal` | Pascal (also Free Pascal/Lazarus) |
| `perl` | Perl |
| `php` | PHP (default server) |
| `php_phpactor` | PHP (Phpactor alternative) |
| `powershell` | PowerShell |
| `python` | Python (default server) |
| `python_jedi` | Python (Jedi alternative) |
| `r` | R |
| `rego` | Rego (Open Policy Agent) |
| `ruby` | Ruby (default server) |
| `ruby_solargraph` | Ruby (Solargraph alternative) |
| `rust` | Rust |
| `scala` | Scala |
| `swift` | Swift |
| `systemverilog` | SystemVerilog |
| `terraform` | Terraform/HCL |
| `toml` | TOML |
| `typescript` | TypeScript (also use for JavaScript) |
| `typescript_vts` | TypeScript (VTS alternative) |
| `vue` | Vue.js |
| `yaml` | YAML |
| `zig` | Zig |

## Language Aliases

- **JavaScript** → use `typescript`
- **C** → use `cpp`
- **Free Pascal / Lazarus** → use `pascal`

## Multi-Language Projects

Specify multiple languages by passing `--language` multiple times:

```bash
serena project create /path --language python --language typescript
```

The first language is the default. Its language server acts as fallback for files not matched by other servers.

## Special Requirements

Some languages require additional setup or installations. Refer to the [Serena language documentation](https://oraios.github.io/serena/01-about/020_programming-languages.html#language-servers) for details.
