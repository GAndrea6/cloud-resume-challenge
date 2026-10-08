# Azure Cloud Resume Challenge

[![Deploy Frontend](https://github.com/GAndrea6/cloud-resume-challenge/actions/workflows/frontend.yml/badge.svg)](https://github.com/GAndrea6/cloud-resume-challenge/actions/workflows/frontend.yml)
[![Deploy Backend](https://github.com/GAndrea6/cloud-resume-challenge/actions/workflows/backend.yml/badge.svg)](https://github.com/GAndrea6/cloud-resume-challenge/actions/workflows/backend.yml)

A serverless online resume on Microsoft Azure, provisioned with Terraform and deployed through GitHub Actions. A visitor counter is stored in Cosmos DB and served by an Azure Function written in Python.

**Live site:** https://icy-tree-0e87ff810.3.azurestaticapps.net

## Architecture

```text
 [ Browser ]
     |  HTTPS
     v
 [ Azure Static Web App ]   serves the HTML / CSS / JS frontend
     |  fetch (CORS)
     v
 [ Azure Function App ]     Python HTTP-triggered API
     |  Cosmos SDK
     v
 [ Azure Cosmos DB ]        NoSQL database storing the visitor count
```

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript on Azure Static Web Apps |
| Backend | Azure Functions (Python 3.10), HTTP trigger |
| Database | Azure Cosmos DB (serverless, NoSQL API) |
| Infrastructure as Code | Terraform |
| CI/CD | GitHub Actions |
| Tests | pytest |
| Observability | Azure Application Insights, KQL queries |

## CI/CD

- `tests.yml` runs the pytest suite on every push and pull request.
- `backend.yml` runs on every push to `main` that touches `backend/`: it installs dependencies, runs the tests, and deploys to the Function App only if they pass. The publish profile is stored as a GitHub Actions secret.
- `frontend.yml` deploys the static site to Azure Static Web Apps.

## Project structure

```text
.
├── .github/workflows/
│   ├── tests.yml           # pytest on every push and pull request
│   ├── frontend.yml        # Frontend deployment pipeline
│   └── backend.yml         # Backend tests and deployment pipeline
├── backend/
│   ├── function_app.py     # Function logic (visitor counter)
│   ├── host.json           # Functions host configuration
│   ├── requirements.txt    # Python dependencies
│   └── test_function.py    # pytest unit tests
├── frontend/
│   ├── index.html
│   ├── style.css
│   └── main.js             # Calls the API and shows the counter
├── infra/
│   ├── main.tf             # Azure resources
│   ├── provider.tf         # Provider configuration
│   ├── variables.tf        # Input variables
│   └── output.tf           # Outputs
└── README.md
```

## Run locally

Backend:

```bash
cd backend
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
func start
```

Tests:

```bash
cd backend
pytest test_function.py
```

Infrastructure:

```bash
cd infra
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

Secrets (Cosmos DB connection, deployment tokens) are never committed: they live in GitHub Actions secrets and in local, git-ignored settings.

## Design decisions

- **Serverless**: Static Web Apps, Functions and Cosmos DB serverless scale with usage, so the site costs very little when idle.
- **Infrastructure as Code**: the Azure environment is described in the `infra/` folder and can be recreated from it.
- **Tests before deployment**: the backend is deployed only if the pytest suite passes.
- **Separate pipelines** for frontend and backend, so a change to one does not redeploy the other.

## What I learned

- Building a full path from browser to database with serverless components.
- Writing unit tests and running them in a pipeline before every backend deployment.
- Keeping secrets out of Git and using GitHub Actions secrets instead.
- Reading Application Insights telemetry with KQL.

## Next improvements

- Store the Terraform state remotely in Azure Blob Storage (it is currently local).
- Add a pipeline running `terraform fmt`, `validate` and `plan` on pull requests.
- Add an architecture diagram and screenshots of the site and the Application Insights dashboard.
