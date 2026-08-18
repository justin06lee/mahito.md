# Skill selection guide

How the orchestrator decides which of the 817 library skills are **applicable** to hardening the project in front of it. This maps observable project signals to families of skill names in `skills-catalog.jsonl`. It is guidance for matching by name/description — it does **not** reproduce any skill's technique.

## Principle

`/mahito` hardens **the user's own application**. So the applicable set is dominated by **defensive, auditing, and hardening** skills, plus offensive skills used *only as review lenses* to locate weaknesses in the user's own code. Skills whose sole purpose is responding to an external compromise, reversing third-party malware, or gathering threat intel on other actors are usually **not applicable** to hardening a codebase — include them only when the project itself is a security/IR/TI tool that implements those workflows.

## Signal → skill family

| Project signal (how to detect it) | Match catalog names containing… |
|---|---|
| Web app / HTTP server / routes, templates, forms | `web`, `xss`, `sql-injection`, `csrf`, `ssrf`, `authentication`, `authorization`, `session`, `cors`, `security-headers`, `input-validation` |
| REST/GraphQL API, gateway, OpenAPI spec | `api`, `graphql`, `rate-limit`, `api-gateway`, `jwt`, `oauth` |
| Auth / login / identity / SSO | `oauth`, `oidc`, `saml`, `jwt`, `session`, `mfa`, `password`, `identity`, `entra`, `active-directory` (only if the app integrates AD/Entra) |
| Secrets in code, `.env`, config, keys | `secret`, `credential`, `vault`, `key-management`, `encryption`, `hardcoded` |
| Cryptography usage (hashing, TLS, signing) | `crypto`, `tls`, `certificate`, `hashing`, `signing`, `random` |
| `Dockerfile` / `docker-compose` | `docker`, `container`, `image`, `dockerfile` |
| Kubernetes manifests / `helm` / `k8s` | `kubernetes`, `k8s`, `rbac`, `pod-security`, `kube-bench` |
| Terraform / CloudFormation / Pulumi / IaC | `terraform`, `iac`, `cloudformation`, `infrastructure`, `cis-benchmark` |
| AWS SDK / ARNs / `aws` config | `aws`, `s3`, `iam`, `cloudtrail`, `cloud` |
| Azure SDK / `az` / Entra | `azure`, `entra`, `azure-active-directory` |
| GCP SDK / `gcloud` | `gcp`, `gcloud`, `iam` |
| Dependency manifests (`package.json`, `requirements.txt`, `go.mod`, `pom.xml`, `Cargo.toml`) | `dependency`, `supply-chain`, `sbom`, `typosquatting`, `vulnerable-dependencies` |
| CI/CD (`.github/workflows`, `.gitlab-ci`, Jenkins) | `ci-cd`, `pipeline`, `devsecops`, `sast`, `dast`, `secret-scanning` |
| Databases / SQL / ORM | `sql-injection`, `database`, `data-protection`, `encryption-at-rest` |
| Mobile app (`android`, `ios`, Swift, Kotlin) | `android`, `ios`, `mobile` |
| Smart contracts (`.sol`, Foundry, Hardhat) | `smart-contract`, `solidity`, `foundry`, `ethereum`, `blockchain` |
| LLM/AI features (prompts, embeddings, MCP, agents) | `llm`, `prompt-injection`, `ai`, `mcp`, `embedding`, `model` |
| Logging / monitoring / SIEM present | `logging`, `audit-log`, `detection`, `monitoring` (harden the app's own logging; skip pure SOC-analyst workflows) |
| Compliance requirement stated (SOC2, HIPAA, PCI, CMMC, GDPR) | `compliance`, `cmmc`, `pci`, `hipaa`, `gdpr`, `nist`, `iso-27001` |

## Always-consider (nearly every project)

- Secrets & credential hygiene
- Dependency / supply-chain risk + SBOM
- Input validation / injection
- Authn/authz correctness
- Transport & at-rest crypto
- Security headers / hardened config
- CI/CD secret scanning + SAST

## Usually skip when hardening an app (include only if the project *is* that kind of tool)

- Live malware reverse engineering (`analyzing-*-malware`, `ghidra`, `volatility`, `cuckoo`)
- Host/disk/memory forensics (`disk-image`, `memory-forensics`, `prefetch`, `registry-artifacts`)
- Threat-intel collection on external actors (`apt-group`, `misp`, `threat-actor`, `leak-site`)
- Active red-team ops against external targets (`c2`, `cobalt-strike`, `phishing-campaign`, `lateral-movement`) — their *knowledge* may still inform a defensive lens, but do not run them against anything you do not own.

## Honesty rule

Whatever you skip, **record it and why** in the final report. Never let "selected a subset" read as "covered everything." If the project scale makes the applicable set huge, cap the fleet per wave and say what was deferred.
