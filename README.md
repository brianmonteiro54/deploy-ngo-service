# Deploy NGO Service

Repositório GitOps com os manifestos Kubernetes do **NGO Service** da plataforma SolidaryTech.

## Visão Geral

Monitorado pelo **ArgoCD**. A tag da imagem em `manifests/deployment.yaml` é atualizada **automaticamente** pelo CI do repositório [`ngo-service`](https://github.com/brianmonteiro54/ngo-service) a cada push na `main`. Não edite a tag manualmente.

Stack: **Python/Flask + PostgreSQL (RDS)** · porta `8081` · namespace `solidarytech-ngo`.

## Manifestos

| Arquivo | Descrição |
|---|---|
| `deployment.yaml` | Deployment (2 réplicas, rolling update, probes) + initContainer que monta a `DATABASE_URL`; sobe via gunicorn |
| `service.yaml` | Service ClusterIP na porta 8081 |
| `ingress.yaml` | Ingress nginx, rota `/ngos` |
| `externalsecret.yaml` | ExternalSecrets: credenciais do RDS + config da app (DB host/port/name) |
| `secretstore.yaml` | SecretStore apontando para o AWS Secrets Manager (us-east-1) |
| `init-db.yaml` | ConfigMap + Job (sync hook wave 1) que cria a tabela `ngos` e popula seeds (idempotente) |

## Pré-requisitos (uma vez)

1. **Infra aplicada** (`terraform apply`): cria o ECR `solidarytech/ngo-service`, o RDS `ngo_db`, os namespaces e o secret `aws-credentials`.
2. **Secret de config no AWS Secrets Manager** chamado `solidarytech/ngo-service` com as chaves:
   - `NGO_DB_HOST` → endpoint do RDS ngo (`terraform output rds_endpoints`)
   - `NGO_DB_PORT` → `5432`
   - `NGO_DB_NAME` → `ngo_db`
3. **UUID do secret do RDS**: em `externalsecret.yaml`, troque `rds!db-REPLACE_WITH_NGO_DB_SECRET_UUID` pelo nome real (Console AWS → Secrets Manager → filtre por `rds!db-`).
4. **App no ArgoCD** apontando para este repositório (`path: manifests`, namespace `solidarytech-ngo`).

> **Acesso público:** `http://solidarytech.pt/ngo` (o ingress reescreve o path singular para a rota real `/ngos` da aplicação). Aponte o DNS de `solidarytech.pt` para o IP/Hostname do Load Balancer do ingress-nginx.
