# helm-charts

Helm charts for the [Bounce suite](https://github.com/trsreagan3/kbouncer):
safety proxies for AI agents and dev workflows.

## Charts

| Chart | Description |
| --- | --- |
| [`kbounce`](charts/kbounce) | Kubernetes safety proxy — gates + audits agent / kubectl traffic to the API server |

## Install

```sh
helm repo add trsreagan3 https://trsreagan3.github.io/helm-charts
helm repo update
helm install kbounce trsreagan3/kbounce
```

The chart pulls its container image from `ghcr.io/trsreagan3/kbounce`.
If that image is private in your environment, configure
`imagePullSecrets` in your values file:

```yaml
imagePullSecrets:
  - name: ghcr-pull
```

## Examples

Each chart ships an `examples/` directory with starter values files
for common deployment shapes:

- [Cooperative mode](charts/kbounce/examples/cooperative-mode-values.yaml)
  — default; agents pass auth headers through, kbounce gates + audits
- [Transparent mode](charts/kbounce/examples/transparent-mode-values.yaml)
  — kbounce terminates client TLS and re-encrypts upstream

## Honest positioning

The Bounce proxies are deterrents, not security boundaries. They gate
and audit traffic that flows through them; they do not stop a caller
with valid credentials from going around the proxy directly. Pair
with Kubernetes RBAC + admission webhooks (for kbounce) or IAM
permissions boundaries (for iam-jit-bouncer) for defense in depth.
See each chart's `NOTES.txt` for the same reminder at install time.

## Releases

Chart releases are cut by the
[chart-releaser](https://github.com/helm/chart-releaser-action)
workflow on every push to `main` that bumps a chart version. The
workflow is **disabled** until the first stable upstream release
lands; see `.github/workflows/release.yml`.

## License

[Apache 2.0](LICENSE).
