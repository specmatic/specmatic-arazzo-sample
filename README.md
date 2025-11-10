# Arazzo Sample Project

This project consists of three microservices:
- **Order API**
- **BFF (Backend for Frontend)**
- **UUID API**

All services are developed using **FastAPI**, **SQLModel**, and **SQLite** as the database system.

## Cloning the Repository

1. Clone the repository
```shell
git clone https://github.com/specmatic/specmatic-arazzo-sample.git
```

2. Initialize and update the specmatic-order-contracts submodule
```shell
git submodule update --init --recursive --remote
```

3. Enable automatic submodule updating when executing git pull
```shell
git config submodule.recurse true
```

## Setup Instructions

### Setting Up Virtual Environment
#### On Unix/macOS:
```shell
python3 -m venv venv
source venv/bin/activate
```

#### On Windows:
```shell
python -m venv venv
venv\Scripts\activate
```

### Installing Dependencies
```shell
pip install -r requirements.txt
```

## Service Interactions
The **[BFF](bff/)** service sends a request to the **[UUID API](uuid_api/)** to retrieve a UUID for a customer based on the request payload.
It subsequently uses this UUID to make a request to the **[Order API](order_api)**.

### Running Contract Tests on the services that will be part of our workflow

Contract tests are run using `Specmatic` and `pytest`.

- To run contract tests across all three services ([BFF](bff/), [UUID API](uuid_api/) and [Order API](order_api)):
  ```shell
  pytest -v -s
  ```
- To run tests for an individual service execute:
  ```shell
  pytest tests/<service_folder> -v -s
  ```

## Authoring Arazzo Workflow Specification

The `workflow/` directory contains a [minimal Arazzo specification](workflow/uuid_order_workflow.arazzo.yaml) which only contains skeleton steps with references to `operations` (using `operationId`) from OpenAPI specifications of each of the services that are part of our workflow.

### Generate the Specification

Specmatic Arazzo can generate a complete Arazzo specification based on the above minimal one by filling in missing parameters, request bodies, and defining success and failure actions and outputs.
To perform the extrapolation, execute:

- On Unix and PowerShell:
```shell
docker run --rm -v "$(pwd):/usr/src/app" specmatic/specmatic-arazzo generate --spec-file=./workflow/uuid_order_workflow.arazzo.yaml -o ./workflow
```

- On Windows CMD Prompt:
```shell
docker run --rm -v "%cd%:/usr/src/app" specmatic/specmatic-arazzo generate --spec-file=./workflow/uuid_order_workflow.arazzo.yaml -o ./workflow
```

After executing this command, you should see two new files generated in the [`workflow/`](workflow/) directory:

1. **Extrapolated Specification:** [`uuid_order_workflow.arazzo_generated.arazzo.yaml`](workflow/uuid_order_workflow.arazzo_generated.arazzo.yaml)
2. **Generated Inputs File:** [`uuid_order_workflow.arazzo_generated.arazzo_input.json`](workflow/uuid_order_workflow.arazzo_generated.arazzo_input.json)

### Validating the Specification

Once the specification is extrapolated, validate it to ensure that all parameters, request bodies, schemas, outputs, and actions are correctly defined.
Run the following command to validate the extrapolated specification:

- On Unix and PowerShell:
```shell
docker run --rm -v "$(pwd):/usr/src/app" specmatic/specmatic-arazzo validate --spec-file=./workflow/uuid_order_workflow.arazzo_generated.arazzo.yaml
```

- On Windows CMD Prompt:
```shell
docker run --rm -v "%cd%:/usr/src/app" specmatic/specmatic-arazzo validate --spec-file=./workflow/uuid_order_workflow.arazzo_generated.arazzo.yaml
```

**Tip:** For testing purposes, consider removing the format of `enum` for `uuidType` field in the [extrapolated Arazzo API spec](workflow/uuid_order_workflow.arazzo_generated.arazzo.yaml).
This alteration should trigger a validation failure (as shown below), demonstrating the effectiveness of the validation process.

```shell
>> ARAZZO-SPEC.WORKFLOW.PlaceOrder.STEP.GetUUID.REQUEST_BODY.PAYLOAD.uuidType
Expected ("Regular" or "Premium" or "Enterprise")
```

### Running the Workflow

Before executing the workflow tests, verify that the input values in the [Arazzo inputs file](workflow/uuid_order_workflow.arazzo_generated.arazzo_input.json) are in line with the seed data specified in `run.py`.
The `productId` in `PlaceOrder` and the `id` in `RetrieveProductDetails` should be set to either `1` or `2`.

#### Initialize Services and Populate Data
Execute the `run.py` script from the root directory to initialize the required services and populate the database with product data:

- On Unix:
```shell
python3 run.py
```

- On Windows:
```shell
python run.py
```

#### Execute Workflow Tests
After initializing the services, run the workflow tests using `Specmatic Arazzo`.

- On Unix and PowerShell:
```shell
docker run --rm -v "$(pwd):/usr/src/app" specmatic/specmatic-arazzo test --serverUrlIndex 1
```

- On Windows CMD Prompt:
```shell
docker run --rm -v "%cd%:/usr/src/app" specmatic/specmatic-arazzo test --serverUrlIndex 1
```

Upon completion of the tests, a detailed HTML report will be generated in the [`build/reports/specmatic/html/index.html`](build/reports/specmatic/html/index.html) directory. 
This report provides a comprehensive overview of the test outcomes, including a workflow diagram and additional information.
