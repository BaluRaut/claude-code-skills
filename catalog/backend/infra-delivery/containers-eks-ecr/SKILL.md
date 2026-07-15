---
name: containers-eks-ecr
description: Container workloads — image hygiene and ECR lifecycle, immutable tags, EKS deployment conventions (probes, resources, secrets), and the deploy/rollback flow.
---

# Containers: ECR & EKS

For the workloads that aren't Lambda [adapt: which services run on EKS].

## 1. Images — small, pinned, scanned

- Multi-stage builds; runtime stage from a slim pinned base
  (`node:20.x-slim` [adapt]) — pruned prod deps only, non-root `USER`
- `.dockerignore` mirrors .gitignore + `.git`, env files — secrets NEVER
  bake into layers (secrets-config; a secret in a layer is leaked even if
  "deleted" in a later layer)
- ECR: scan-on-push ON, criticals block the pipeline [adapt policy]

## 2. ECR tags & lifecycle

- **Immutable tags, tagged by git SHA** (+ optional semver) — `latest` in
  a K8s manifest fails review: it makes rollback and "what's running?"
  unanswerable
- Lifecycle policy per repo: keep last N [adapt: ~20] + release tags,
  expire untagged in days — unbounded ECR is a silent bill

## 3. EKS manifests — every pod declares its contract

[adapt: raw manifests / Helm / Kustomize — follow the repo]

- **Resources**: requests AND limits set from measurement — no requests =
  unschedulable chaos under pressure
- **Probes**: liveness (process dead → restart) ≠ readiness (not ready →
  no traffic); readiness must NOT depend on downstreams you can't fix by
  restarting (a DB blip flapping every pod is self-inflicted)
- ≥2 replicas + PodDisruptionBudget for anything user-facing;
  `securityContext`: runAsNonRoot, no privilege escalation
- Config via ConfigMap; secrets via [adapt: External Secrets Operator →
  Secrets Manager — same source of truth as Lambda, not a parallel store]

## 4. Deploy & rollback

- Pipeline: build → push SHA tag → update manifest ref [adapt: GitOps/
  helm upgrade] — deploys are a REF CHANGE, so rollback is re-pointing to
  the previous SHA, boring by design
- `kubectl` by hand in prod = incident response only, logged, followed by
  making the manifests match reality

## 5. Verify

Image runs as non-root locally; kill a pod in staging (self-heals, no
5xx thanks to readiness+replicas); roll back one release in staging and
confirm it's one ref change; ECR shows scan results and lifecycle pruning.
