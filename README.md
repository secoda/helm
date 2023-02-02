### Security notice

Running this terraform will generate state files with sensitive environment variables. These must not be checked into source control. We encourage using Hashicorp Cloud to store state files.

# Secoda On-Premise (Google Cloud, Terraform)

## Initial Steps

1. `brew install --cask google-cloud-sdk`

2. `gcloud init && gcloud components install gke-gcloud-auth-plugin`

3. `gcloud auth application-default login`

4. Get the repository:

```
git clone https://github.com/secoda/terraform-gcp-secoda
cd terraform-gcp-secoda
brew install terraform
terraform init
```

## Simple deployment

1. `cp rename.onprem.tfvars onprem.tfvars` then fill `onprem.tfvars` in:

```bash
docker_password="*****"
region="us-east1"
```

2. Then run:
```bash
# The order of these commands is important:
terraform apply -var-file="onprem.tfvars" --target=google_container_node_pool.nodes
# Type `yes` at the prompt.
terraform apply -var-file="onprem.tfvars"
# Type `yes` at the prompt.
```

3. You must create a CNAME record with your DNS provider that points your your domain, i.e. `secoda.yourcompany.com` to your ingress external ip.
4. Wait about 10 minutes. Then open `https://secoda.yourcompany.com` to test out the service. It will only listen on **HTTPS**. Make sure you use `https://` and not `http://`.
5. We suggest using _Cloudflare ZeroTrust_ to limit access to Secoda; optional.

## Connecting to Secoda

- Load balancer is publicly accessible by default (IP is returned after running `terraform apply`). You will not be able to connect to the IP. The ingress will only accept connections via the `domain` name. There will be a delay on first setup as the registration target happens ~5 minutes.
- We suggest using _Cloudflare ZeroTrust_ to limit access to Secoda.

## Updating Secoda

- It is configure to automatically pull the latest images on restart.
- `kubectl rollout restart deployment secoda-web` will redeploy the application with the latest images.

## Destroying the deployment (irreversible)

1. Then run:
```bash
terraform destroy -var-file="onprem.tfvars"
# Type `yes` at the prompt.
```

# Misc.

## Hashicorp Cloud

To store state in Hashicorp cloud, which we recommend, please complete the following steps. You should be a member of a _Terraform Cloud_ account before proceeding.

In this directory, run `terraform login`. In `versions.tf` please uncomment the following lines and replace `secoda` with your organization name.

```yaml
backend "remote" {
  organization = "secoda"
}
```
