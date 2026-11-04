# n8n service for Kubernetes on Wodby

Run n8n as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for n8n.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [n8n application stack](https://github.com/wodby/stack-n8n)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `n8n` |
| Type | Application service |
| Versions | `2` by default |
| Workloads | `main` (StatefulSet, primary) |
| Containers | `n8n` using `n8nio/n8n`; `runner` using `n8nio/runners` |
| Endpoints | `n8n`: HTTP 5678 (main) |
| Service links | PostgreSQL (`postgres`), required; SMTP server (`sendmail`) |
| Volumes | `data` (10 GB) |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateful`; version `0.4.0` |

## Use this service

Use this service through [n8n application stack](https://github.com/wodby/stack-n8n), or reference `n8n` from a custom
Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Owner account

n8n applies the owner's email and password from the service's required Owner
email and Owner password settings on every start, so n8n's setup page is never
open to the public. To change them, change the settings and redeploy; n8n
doesn't let you edit them in its own settings. The password needs at least 8
characters.

## Encryption key

n8n encrypts stored credentials with a key Wodby generates for the service.
Keep it: a different key makes saved credentials unreadable.

## Upgrades

n8n migrates its database when a newer version starts. Back up the PostgreSQL
database before an upgrade; moving back to an older version is not supported.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
