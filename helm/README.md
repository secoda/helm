# Helm Chart (Enterprise)

## Disclaimer

### New customers

Please contact us for your company-specific Docker token. Secoda on-premise is an enterprise product, and we are happy to provide you with a token if you contact us through Intercom or Slack.


## Prerequisites

- This chart requires **Helm 3.0**.
- A PostgreSQL database.
        - We recommend using an externally managed postgres database with backups enabled (for example, AWS RDS) as persistent volumes are not as reliable.
        - This chart runs Secoda in a single pod and requires about *4 vCPU* and *16 GB memory*.

## Setup

Once your database is created, connect to it and then create the keycloak user and two seperate databases on it.

```bash
psql -h <HOST> -U postgres
```

```bash
create database keycloak;
create database secoda;
create user keycloak with encrypted password '<SAME_PASSWORD_AS_DEFINED_IN_PREDEFINED-SECRETS.YAML>';
grant all privileges on database keycloak to keycloak;
grant all privileges on database secoda to keycloak;
```

## Usage

1.  Run this command `git clone https://github.com/secoda/secoda-helm.git`

2.  Modify the `examples/predefined-secrets.yaml` file. Replace all values that have a comment next to them.

- Uncomment `ingress.hosts` and change `ingress.hosts.host` to be the hostname where you will access Secoda.
- Load Balancer TLS is required for Secoda, uncomment `ingress.tls` and:
  - Specify the name of the SSL certificate to use as the value of `ingress.tls.secretName`.
  - Specify the hostname where you will access Secoda (the same value you configured for `ingress.hosts.host`).

GKE-specific configurations:

- Specify `/*` as the value of `ingress.hosts.paths.path`.
- The field `ingress.tls.servicePort` is not required.
- (Optional) Follow SQL Auth Proxy [guide](https://cloud.google.com/sql/docs/postgres/connect-kubernetes-engine) and enable `cloudSqlAuthProxy.enabled` and modify `cloudSqlAuthProxy.databaseName`.

3.  Add your customer-specific docker secret, this is required to pull Secoda's images:

        $ kubectl create secret docker-registry secoda-dockerhub --docker-server=https://index.docker.io/v1/ --docker-username=secodaonpremise --docker-password=<CUSTOMER_SPECIFIC_TOKEN_PROVIDED_BY_SECODA> --docker-email=carter@secoda.co

4.  Now you're all ready to install Secoda:

        $ gcloud container clusters get-credentials <CLUSTER> --region <REGION> # If using GKE.
        $ cd helm/helm/charts
        $ helm install -f secoda/examples/predefined-secrets.yaml secoda ./secoda
        
## Contributing

Install these tools:

```bash
brew install pre-commit
pre-commit install
```

## Troubleshooting

If you are getting the status error `ImagePullBackOff`. It usually has to do with Docker access token issues. Check the following:
* `imagePullSecrets` must all be in the same namespace as the Pod.
* Confirm the docker secret has been created: `kubectl get secrets` and `kubectl get secret secoda-dockerhub`
