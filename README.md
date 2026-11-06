# Suliman Khan — Project Portfolio

**Backend & Security Engineer** · Python / Go · AWS & Kubernetes · Cloud-Native & LLM Application Security

📧 **sulmn.cs@gmail.com** · [LinkedIn](https://linkedin.com/in/suliman-khan-2b9280251) · [Website](https://suliman-khan-security.netlify.app/) · Islamabad, Pakistan

---

This repository shows the projects I have designed and built: what each one does, how it works, and the results I measured. Every number on these pages comes from my own test runs, benchmarks or lab experiments, and each page states the project's limits honestly.

> **Why the code isn't here:** to protect my work, the source code of most projects is kept in private repositories. I'm happy to **walk through the code or give a live demo** in an interview. Just email me.

## Projects

| Project | What it does | Main tech | Status |
|---|---|---|---|
| **[LLM Application Security Gateway (AI Firewall)](projects/llm-ai-firewall/)** | A security proxy in front of LLMs that stops prompt injection, jailbreaks, system-prompt theft and secret leaks, using a cost-ordered chain of detectors | Python, FastAPI, XGBoost, DeBERTa-v3, Llama judge, Redis | v1.0 complete (lab) |
| **[AgentGuard](projects/agentguard/)** | Scores every website visitor 0–100 and separates humans, bots and cryptographically verified AI agents (Web Bot Auth, RFC 9421) | Python, FastAPI, Redis, PostgreSQL, React, Docker | Working personal project |
| **[LLM Red-Team Toolkit](projects/llm-redteam-toolkit/)** | The offensive counterpart to the AI Firewall: fires prompt-injection and jailbreak attacks at a model, grades each by OWASP LLM category and severity, and writes HTML/JSON reports | Python, Typer, FastAPI, Ollama, pytest | Working learning project |
| **[Adaptive Network Intrusion Detection System](projects/adaptive-network-ids/)** | My final-year thesis: a live network IDS combining payload signatures, heuristics, XGBoost and anomaly triage, with an analyst dashboard and human-in-the-loop retraining | Python, FastAPI, Scapy, XGBoost, Isolation Forest, React | Thesis prototype · paper in preparation |
| **[Kernel-Level Runtime Security Platform](projects/runtime-security-platform/)** | Watches Kubernetes containers at the Linux-kernel level with eBPF, scores activity with ML, and kills malicious processes in the kernel | Go, Python, Tetragon eBPF, Kafka, k3s, React | Working end to end (k3s lab) |
| **[SLSA Level 3 Supply-Chain Security Platform](projects/supply-chain-security-platform/)** | Signs every container image keylessly, attaches provenance and SBOMs, and blocks unsigned or tampered images at admission | GitHub Actions, Sigstore Cosign, Syft, Grype, Kyverno | v1.0 complete |
| **[Purple-Team Detection-as-Code](projects/purple-team-detection-as-code/)** | A CI pipeline that attacks itself on every rule change and fails the pull request if a detection doesn't fire | Sigma, Falco, GitHub Actions, MITRE ATT&CK | Complete · [code is public](https://github.com/Suliman8/purple-team-detection-as-code) |
| **[GitOps-Native Zero-Trust Kubernetes](projects/gitops-zero-trust/)** | Cryptographic workload identity, mutual TLS and identity-based access, all managed from Git | ArgoCD, SPIFFE/SPIRE, Istio, Vault, Go | In progress (weeks 1–6 of 10 done) |

**Also published:** [LabFlow-NIDS](https://www.kaggle.com/datasets/sulimankhan0056/labflow-nids-flow-based-network-intrusion-dataset) on Kaggle, an open network-intrusion dataset of 873,080 labelled flows that I generated in my own Docker lab.

## What each page covers

- **The problem** the project solves
- **What I built** and the architecture diagram
- **Key features** and **design decisions**, meaning why I chose one approach over another
- **Results**, the verified numbers and how each was measured
- **Honest limits**, what is not done yet

## Contact

Interested in a code walkthrough or a live demo? Email **sulmn.cs@gmail.com** or message me on [LinkedIn](https://linkedin.com/in/suliman-khan-2b9280251).

<sub>© 2026 Suliman Khan. All rights reserved. See [LICENSE](LICENSE). No part of this repository may be copied or reused without permission.</sub>
