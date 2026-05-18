# kbounce

Kubernetes safety proxy for AI agents and dev workflows.

kbounce sits between agent / kubectl traffic and the Kubernetes API
server. It gates requests against named profiles, audits everything
that flows through, and returns a clean denial when a request falls
outside the active profile.

This chart deploys kbounce as a Deployment + Service in your cluster.

## Install

```sh
helm repo add trsreagan3 https://trsreagan3.github.io/helm-charts
helm install kbounce trsreagan3/kbounce
```

For private GHCR images, set `imagePullSecrets`:

```yaml
imagePullSecrets:
  - name: ghcr-pull
```

## Modes

kbounce ships two modes, both first-class:

- **cooperative** (default): callers pass their own auth headers
  through to the upstream API server. kbounce gates + audits but does
  not terminate TLS. Easiest to drop in.
- **transparent**: kbounce terminates client TLS and re-encrypts to
  the upstream. Requires you to issue kbounce its own server cert and
  point clients at the kbounce service.

See `examples/cooperative-mode-values.yaml` and
`examples/transparent-mode-values.yaml`.

## Key values

| Key | Default | Description |
| --- | --- | --- |
| `image.repository` | `ghcr.io/trsreagan3/kbounce` | Container image |
| `image.tag` | `main` | Image tag (pin in production) |
| `proxy.mode` | `cooperative` | `cooperative` or `transparent` |
| `proxy.port` | `8443` | Wire listener port |
| `proxy.defaultPolicy` | `deny` | Fail-closed default |
| `profiles.active` | `full-user` | Default profile (lean-permissive) |
| `profiles.additionalProfiles` | `[]` | Extra profiles rendered into a ConfigMap |
| `upstream.apiserver` | `https://kubernetes.default.svc` | Upstream target |
| `mgmt.port` | `8444` | Healthz + metrics listener |
| `tls.enabled` | `false` | Terminate TLS at kbounce (required for transparent mode) |
| `versionCheck.enabled` | `false` | Opt-in GitHub version check (off by default) |
| `rbac.enabled` | `false` | kbounce does not need RBAC by default |

See `values.yaml` for the full set with inline documentation, and
`values.schema.json` for a JSON Schema that `helm install --validate`
checks against.

## Defaults

- Non-root distroless UID `65532`
- `readOnlyRootFilesystem: true` with an emptyDir at `/tmp` for SQLite
  scratch
- Liveness + readiness probes against `/healthz` on the mgmt port
- Cooperative mode + `full-user` profile per the Bounce default
  profile pattern (lean-permissive, audit-everything, block only
  universally-dangerous ops)

## Honest positioning

kbounce is a deterrent proxy, not a security boundary. Anyone who can
reach the API server directly with valid credentials can bypass
kbounce. Pair with Kubernetes RBAC and admission webhooks for defense
in depth. The proxy's value is fast, configurable gating + visible
audit for traffic that *does* flow through it.
