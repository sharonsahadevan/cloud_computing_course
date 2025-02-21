# Show Function as a Service

The goal: show Azure Function is connections with other services

## Create Azure components

1. Login to azure `az login` and select Subscription `az account set -s $SUBSCRIPTION_ID`

2. Change working directory to: `cd src/terraform/06-core-services-faas`

3. Create `terraform.tfvars` file. This file contains your unique properties for the rest of terraform configuration. You can use a sample generator script: `../generate_tfvars.sh`

4. Review proposed terraform configuration:

  - Event Hub (e.g. azure-kafka)
  - Storage Account as object store
  - FaaS itself integrated with two above

5. Provision Azure components via terraform

```sh
terraform init
terraform apply
```

6. Parse terraform output to environment variables that will be picked up by azure-function locally:

```sh
export AzureWebJobsStorage=$(terraform output -raw azure_web_jobs_storage)
export eventHubName=$(terraform output -raw event_hub_name)
export CloudComputingEventHubConnectionString=$(terraform output -raw event_hub_connection_string)
export StorageAccountConnectionString=$(terraform output -raw storage_account_connection_string)
export StorageAccountContainerName=$(terraform output -raw storage_account_containername)
export FUNCTION_APP_NAME=$(terraform output -raw azure_function_name)
```

## Deploy the function

1. Change working directory to function sources: `cd ../../az_func`

2. Create and activate virtual environment

```sh
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

3. Run locally

    1. Run the application: `func start --python`.

    2. Insert events **from another terminal**: `echo $(date) | xargs -I{} curl -d '{"name":"the first", "time":"{}"}' -X POST http://localhost:7071/api/HttpToEventHub`.

4. Deploy function to Azure: `func azure functionapp publish $FUNCTION_APP_NAME --python`. Note, it takes _some time_ to finish the process.

5. Ingest events

```sh
# Get function code
func azure functionapp list-functions $FUNCTION_APP_NAME --show-keys

# Get function URL from previous command or from Code+Test view, for example:
FUNCTION_URL=https://cloudcomp-ex4f-func.azurewebsites.net/api/HttpToEventHub?code=Q3NoO2M4LmARBdh/nAcRe0Giaa0yKrVBMN6bbQQZ9ZaVxqyvBXRXKg==
echo $(date) | xargs -I{} curl -d '{"name":"the forth", "time":"{}"}' -X POST $FUNCTION_URL
```

6. Review and Feedback:

    - Review setup process
    - Consider failure strategies
    - Propose mitigations: poison queue, jitter, retries

## Clean up

Remove created resources: `cd - && terraform destroy`
