# mcl — mycel platform enabler

CLI and daemon for managing mycel infrastructure, modules, and identity.

## Install

Download the binary for your platform from [Releases](https://github.com/CogniWorks/mycel_cli/releases):

```bash
# macOS Apple Silicon
curl -L https://github.com/CogniWorks/mycel_cli/releases/latest/download/mcl-darwin-arm64 -o mcl

# macOS Intel
curl -L https://github.com/CogniWorks/mycel_cli/releases/latest/download/mcl-darwin-amd64 -o mcl

# Linux x86_64
curl -L https://github.com/CogniWorks/mycel_cli/releases/latest/download/mcl-linux-amd64 -o mcl

# Linux ARM64
curl -L https://github.com/CogniWorks/mycel_cli/releases/latest/download/mcl-linux-arm64 -o mcl
```

```bash
chmod +x mcl && sudo mv mcl /usr/local/bin/
```

Or build from source:

```bash
GOWORK=off go build -ldflags="-s -w" -trimpath -o bin/mcl ./cmd/mcl/
```

## Usage

```
mcl <command> [flags]
```

### Management

| Command | Description |
|---------|-------------|
| `mcl init` | Bootstrap a workspace |
| `mcl config set/get/list` | Read and write configuration |
| `mcl serve --port 9417` | Start daemon (REST + MCP on same port) |

### Fabric (Infrastructure)

| Command | Description |
|---------|-------------|
| `mcl fabric up --env dev` | Start all providers |
| `mcl fabric down --env dev` | Stop all providers |
| `mcl fabric status --env dev` | Health check with per-service detail |
| `mcl fabric logs --name mycel_app_db` | Stream container logs |
| `mcl fabric env list` | List environments |
| `mcl fabric env switch --env staging` | Switch active environment |
| `mcl fabric secret set/get/list/rotate` | Manage secrets |
| `mcl fabric doctor` | Diagnose infrastructure |

Providers: `mycel_app_db` → `mycel_vault` → `mycel_temporal` → `mycel_idp` → `mycel_monitoring`

### Module

| Command | Description |
|---------|-------------|
| `mcl module new --name mycel_foo` | Scaffold a new module |
| `mcl module build --name mycel_iam` | Build Docker image |
| `mcl module test --name mycel_iam` | Run tests |
| `mcl module release --name mycel_iam` | Tag, build, push |
| `mcl module scan --name mycel_iam` | Security scan |

### IAM

| Command | Description |
|---------|-------------|
| `mcl iam provision --env dev` | Create schema, seed roles, generate keys |
| `mcl iam reset --env dev` | Drop and recreate (dev only) |
| `mcl iam status --env dev` | Verify IAM health |

## Architecture

### Pipeline

Every operation runs through a three-tier pipeline:

1. **Execute & Check** — run the command, verify post-conditions
2. **Recipe** — if checks fail, match against known fix patterns, apply fix, retry once
3. **Failure Report** — structured report with context if still failing

Results carry three independent axes:

| Axis | Question | Example |
|------|----------|---------|
| **Execution** | Did the command run? | exit 0, checks pass |
| **Output** | What data came back? | service listing |
| **Assessment** | What does it mean? | `unhealthy — 0/12 running` |

### Daemon

`mcl serve` exposes REST and MCP on the same port:

- `POST /v1/ops/{command}` — start async operation
- `GET /v1/ops/{id}` — get result
- `GET /v1/events?op={id}` — SSE stream
- `POST /mcp` — JSON-RPC 2.0 (MCP protocol)
- `GET /health` — health check
- `GET /swagger/` — API docs

### Locking

Operations declare lock requirements (`fabric:{env}`, `module:{name}`). Read locks allow concurrency; write locks are exclusive. Prevents conflicting operations on the same resource.

## Testing

```
GOWORK=off go test ./... -v
```

66 integration tests covering pipeline contracts, REST/MCP endpoints, broadcaster, lock manager, and CLI parsing. No Docker required.

## Release

### 1. Build

Local dev build:

```bash
GOWORK=off go build -ldflags="-s -w" -trimpath -o bin/mcl ./cmd/mcl/
```

Cross-compile release binaries:

```bash
GOWORK=off GOOS=darwin GOARCH=arm64 go build -ldflags="-s -w" -trimpath -o dist/mcl-darwin-arm64 ./cmd/mcl/
GOWORK=off GOOS=darwin GOARCH=amd64 go build -ldflags="-s -w" -trimpath -o dist/mcl-darwin-amd64 ./cmd/mcl/
GOWORK=off GOOS=linux  GOARCH=arm64 go build -ldflags="-s -w" -trimpath -o dist/mcl-linux-arm64  ./cmd/mcl/
GOWORK=off GOOS=linux  GOARCH=amd64 go build -ldflags="-s -w" -trimpath -o dist/mcl-linux-amd64  ./cmd/mcl/
```

| Flag | Effect |
|------|--------|
| `-s` | Strip symbol table |
| `-w` | Strip DWARF debug info |
| `-trimpath` | Remove local filesystem paths from binary |

Result: ~19MB per binary (vs ~24MB without stripping).

### 2. Tag and publish

```bash
git tag v0.x.0
git push origin v0.x.0
gh release create v0.x.0 dist/mcl-* --title "v0.x.0" --notes "Release notes"
```

This creates a GitHub release with the 4 platform binaries attached. Requires [`gh` CLI](https://cli.github.com/) (`brew install gh`).

### 3. Source archive protection

GitHub automatically attaches source archives (tar.gz, zip) to every release. These are generated via `git archive`, which respects `.gitattributes` `export-ignore` rules.

Our `.gitattributes` excludes:

| Pattern | What it strips |
|---------|---------------|
| `/cmd/` | CLI entrypoint source |
| `/internal/` | All internal packages |
| `/docs/` | Swagger generated docs |
| `/bin/`, `/dist/` | Build artifacts |
| `/test/`, `/test_workspace/` | Test infrastructure |
| `go.mod`, `go.sum` | Dependency manifests |
| `*_test.go` | Test files |

The source archives that ship with the release contain only: `README.md`, `LICENSE`, `Makefile`, `.gitattributes`. No source code is exposed. Users get binaries; source stays private.

## License

PROPRIETARY — CogniWorks. All rights reserved.
