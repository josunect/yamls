# Repository Guidance

## Purpose

This repository contains YAML examples used to configure Kiali-related environments, proxies, observability integrations, and test setups. Most files are Kubernetes or OpenShift manifests; a smaller subset are Helm values files.

## Layout

- `openshiftMonitoring/`: OpenShift monitoring, Kiali, Prometheus Operator, and Istio telemetry resources.
- `basicAuthProxy/`: example basic-auth proxy manifests plus usage notes.
- `grafanaProxy/`: Grafana proxy manifests plus manual setup steps in the README.
- `k8sgateway/`: Gateway API resources.
- `tempo/`: Helm values for Tempo, not plain `kubectl apply` manifests.
- Root-level YAML files: small RBAC and Istio telemetry examples.

## Editing Rules

- Preserve the existing manifest style in each file. Do not rename `.yml` files to `.yaml` or the reverse unless the user asks.
- Keep namespaces, `apiVersion`, and `kind` aligned with the surrounding resources. Many examples intentionally target `istio-system` and OpenShift-oriented setups.
- Treat `tempo/tempo-grpc-helm.yaml` as a Helm values file, not a Kubernetes manifest. Do not add `apiVersion`/`kind` there unless the user explicitly wants to convert it.
- Avoid committing real credentials, tokens, certificates, or cluster-specific secrets. Use placeholders or clearly fake example values.
- When changing proxy behavior in `grafanaProxy/`, keep `grafanaProxy/README.md` in sync because the deploy flow depends on external secrets such as `grafana-proxy-tls` and the optional `grafana-bearer-token`.
- Prefer minimal, targeted edits. This repo is mostly example configuration, so preserving readability is more important than large refactors.

## Validation

- For Kubernetes/OpenShift manifests, prefer lightweight validation such as `kubectl apply --dry-run=client -f <file>` when practical.
- For Helm values files, validate structure manually unless the user asks for chart-specific validation.
- If you cannot run validation, say so clearly in the final response.
