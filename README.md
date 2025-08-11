### AI Agent–Driven Website Provisioning: End-to-End Design, Runbook, and Integration Guide

## Executive summary
- Build a Python FastAPI agent to scaffold from templates, open a PR tagging DevOps, on merge create Jenkins job, verify K8s rollout, and apply Ingress.
- Integrations: GitHub (MCP or REST), Jenkins, Kubernetes; simple UI and stateful workflow.
- Includes architecture, flow, API, templates, security, runbook, and rollout steps.

## High-level workflow
- User submits: domain, app type (frontend/backend), template.
- Agent: PR from template → DevOps approval → Jenkins build → K8s health → Ingress → live URL.

## Architecture diagram (Mermaid)


## Components
- Web UI: minimal form for domain/app type/template, status page.
- Agent Orchestrator: Python FastAPI, background tasks or RQ/Celery.
- GitHub: MCP server preferred; fallback to REST (PyGithub).
- Jenkins: REST API (crumb + token).
- Kubernetes: Python client (rollout checks, apply Ingress).
- Templates: app templates in GitHub; Ingress Jinja2 templates.
- State/queue: start simple; can evolve to SQLModel/Postgres and Redis/RQ.

## API contract (FastAPI)
- POST : body { domain, app_type, template_id } → returns { provision_id, status }
- GET : returns current status and links (PR URL, build URL, live URL)
- POST : handles pull_request; on merged, continues pipeline

## Configuration and secrets
- GitHub: GITHUB_TOKEN, GITHUB_OWNER, GITHUB_REPO, GITHUB_DEVOPS_TEAM_SLUG, MCP_GITHUB_SERVER_ENDPOINT, WEBHOOK_SECRET
- Jenkins: JENKINS_URL, JENKINS_USER, JENKINS_TOKEN
- Kubernetes: KUBECONFIG or in-cluster; K8S_NAMESPACE_DEFAULT, K8S_INGRESS_CLASS
- Templates: TEMPLATE_REPOS or TEMPLATE_ROOT, INGRESS_TEMPLATES_PATH

## GitHub integration (MCP first, REST fallback)
- MCP tools: create branch, write files, open PR, request review, add labels.
- REST fallback (PyGithub): branch from default → commit template → open PR → add labels → request review (DevOps team).
- Webhook: verify HMAC; on , resume pipeline.

## Jenkins integration
- Ensure folder and job exist (Pipeline or Freestyle via config.xml).
- Trigger build with parameters; poll queue → build number → result.

## Kubernetes integration
- Wait for Deployment rollout (Available=True, ready replicas == desired).
- Apply Ingress via NetworkingV1 using Jinja2 template.

## Ingress template (example)


## Dependencies (Python)
- fastapi, uvicorn, requests, PyGithub, Jinja2, kubernetes
- optional: SQLModel, psycopg, redis, rq

## Operational runbook
- PR not opening: check token scopes, repo protection, MCP availability.
- PR merged but no build: check webhook delivery and Jenkins creds.
- Build failed: inspect Jenkins console; fix template or pipeline; retry.
- Deployment unhealthy: inspect Pods/Events; roll back image.
- Ingress unreachable: check DNS, TLS secret, ingress controller logs.

## Publishing & PDF
- GitHub Wiki: copy this README into wiki or .
- PDF: print from browser or .
