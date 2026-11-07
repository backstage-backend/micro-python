# micro-python

Prueba de integración entre Backstage y GitHub

## API

The service exposes an in-memory REST API for `items`. The OpenAPI contract is available at [`docs/openapi.yaml`](docs/openapi.yaml).

| Method | Path | Description |
| --- | --- | --- |
| `GET` | `/api/v1/items` | List all items |
| `POST` | `/api/v1/items` | Create an item |
| `GET` | `/api/v1/items/{id}` | Get an item |
| `PUT` | `/api/v1/items/{id}` | Replace an item |
| `DELETE` | `/api/v1/items/{id}` | Delete an item |

An item requires a non-empty `name` and accepts an optional `description`. Data is stored in memory and is lost when the service restarts.

## CI configuration

The CI workflow supports an optional SonarCloud scan. To enable it, configure
the repository or organization secret `SONAR_TOKEN` and the repository variable
`SONAR_ENABLED` with the value `true`. The scan is skipped otherwise, so builds
do not fail when SonarCloud credentials are not available. The SonarCloud URL
is set explicitly to `https://sonarcloud.io`.

The scan uses the GitHub repository as the SonarCloud project key and
`backstage-backend` as the SonarCloud organization. If your SonarCloud
organization is different, update `sonar.organization` in the generated
workflow.

## AWS deployment

The selected deployment option adds Terraform under `terraform` and uses the
common `.github/workflows/cd.yml` workflow for deployment.

Before running the workflow, configure the `aws-infrastructure` environment
with `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and these variables:
`AWS_REGION`, `TF_STATE_BUCKET`, `API_GATEWAY_NAME`, and
`EKS_CLUSTER_NAME`. Only the variables needed by the selected deployment are
used. The AWS identity must be allowed to deploy Lambda/API Gateway resources
or to access the configured EKS cluster.

Lambda services use the existing REST API Gateway and expose the service under
`/<service-name>/{proxy+}`. Terraform discovers the existing REST API by the
configured `API_GATEWAY_NAME`, so no account resource ID is stored in the
generated repository. Because the API has no stage, Terraform creates the
shared `prod` stage automatically. The resulting URL is
`/prod/<service-name>/{proxy+}`.

Kubernetes services deploy to the EKS cluster configured in
`EKS_CLUSTER_NAME` and expose an internet-facing ALB path under
`/<service-name>/*`. The EKS cluster must have the AWS Load Balancer Controller
installed. Set `container_image` only when deploying outside the generated
workflow.

## Local development

### Node.js

```bash
npm install
npm start
```

### Python

```bash
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
python src/app.py
```