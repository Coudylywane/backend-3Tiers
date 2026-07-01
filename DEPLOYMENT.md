# Deploiement AWS avec Terraform et GitHub Actions

Ce projet construit l'image Docker du backend, la pousse sur Docker Hub, puis utilise Terraform pour creer une instance EC2 AWS. L'instance lance Docker Compose avec deux services:

- `backend`: l'image Docker construite par GitHub Actions
- `db`: PostgreSQL 15 avec un volume Docker persistant

## Secrets GitHub obligatoires

Dans `Settings > Secrets and variables > Actions > Secrets`, ajouter:

- `DOCKERHUB_USERNAME`: nom d'utilisateur Docker Hub
- `DOCKERHUB_TOKEN`: token Docker Hub
- `AWS_ACCESS_KEY_ID`: access key AWS
- `AWS_SECRET_ACCESS_KEY`: secret key AWS
- `AWS_REGION`: region AWS, par exemple `eu-west-3`
- `TF_STATE_BUCKET`: bucket S3 qui stocke l'etat Terraform
- `TF_STATE_KEY`: chemin du state, par exemple `tp-3tiers/backend/terraform.tfstate`
- `DB_PASSWORD`: mot de passe PostgreSQL utilise sur l'instance

Le bucket S3 doit exister avant le premier lancement du workflow.

## Variables GitHub optionnelles

Dans `Settings > Secrets and variables > Actions > Variables`, tu peux ajouter:

- `DB_USER`: utilisateur PostgreSQL, defaut `tpuser`
- `DB_NAME`: base PostgreSQL, defaut `tpdb`
- `EC2_KEY_NAME`: nom d'une key pair EC2 existante pour SSH
- `SSH_CIDR`: IP/CIDR autorisee en SSH, defaut `0.0.0.0/0`
- `EC2_INSTANCE_TYPE`: type EC2, defaut `t2.micro`
- `PROJECT_NAME`: prefixe des ressources AWS, defaut `tp-3tiers-backend`

## Lancement

Quand tu pushes sur la branche `main`, `.github/workflows/ci.yml`:

1. build l'image Docker du backend
2. push l'image sur Docker Hub avec le tag `${github.sha}`

Quand le workflow CI reussit, `.github/workflows/cd.yml` demarre automatiquement:

1. recupere le SHA du workflow CI termine
2. reconstruit le nom de l'image Docker Hub avec ce tag
3. lance `terraform init`, `terraform validate`, puis `terraform apply`
4. deploie cette image precise sur l'instance EC2

Tu peux aussi lancer le CD manuellement avec `workflow_dispatch` en donnant un `image_tag` existant.

L'URL publique du backend est exposee par Terraform via l'output `backend_url`.
