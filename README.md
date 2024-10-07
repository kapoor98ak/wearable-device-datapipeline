
# Azure DevOps Build and Release Pipeline Project

## Project Overview

This project aims to automate the build and release processes for a data engineering application using Azure DevOps. It consists of creating a build pipeline that compiles code, runs tests, and produces artifacts, as well as a release pipeline that deploys the artifacts to a target environment (Databricks).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Build Pipeline](#build-pipeline)
    - [Creating the YAML File](#creating-the-yaml-file)
    - [Defining the Pipeline Steps](#defining-the-pipeline-steps)
- [Release Pipeline](#release-pipeline)
    - [Creating the Release Pipeline](#creating-the-release-pipeline)
    - [Configuring Variables](#configuring-variables)
- [Integration Testing](#integration-testing)
- [Bash Script for Integration Testing](#bash-script-for-integration-testing)
- [Conclusion](#conclusion)

## Prerequisites

Before you begin, ensure you have the following:

- An Azure DevOps account and access to create pipelines.
- A Databricks workspace for deploying the application.
- Necessary permissions to create branches, pull requests, and manage pipelines.

## Project Structure

The project contains the following components:

- **YAML Files**: Configuration files for the build pipeline.
- **Bash Scripts**: Scripts for executing tasks in the release pipeline.
- **Notebooks**: Jupyter notebooks containing the data engineering logic.

## Build Pipeline

### Creating the YAML File

1. Log in to your Azure DevOps portal.
2. Go to your project and navigate to **Repos**.
3. Create a new branch from the **release branch**.
4. Create a new file and name it `azure-pipelines.yml`.
5. Paste the YAML content that defines your build pipeline.

### Defining the Pipeline Steps

The YAML file contains the following key components:

- **Trigger**: Configured to run on all branches and only trigger one build when multiple commits occur simultaneously.
- **Agent**: Specifies `ubuntu-22.04` as the virtual machine to be used.
- **Steps**: Includes tasks such as installing Python, installing required tools, checking out the latest code, and archiving files into a zip for artifacts.

## Release Pipeline

### Creating the Release Pipeline

1. Navigate to **Pipelines** > **Releases** in Azure DevOps.
2. Click on **New Release Pipeline** and select the **Empty Job** template.
3. Define the artifacts by selecting the build pipeline created earlier as the source.

### Configuring Variables

1. Go to the **Variables** tab in your release pipeline.
2. Add environment variables:
   - `DATABRICKS_HOST`: The URL of your Databricks workspace.
   - `DATABRICKS_TOKEN`: The access token for authentication.

## Integration Testing

The release pipeline includes an integration testing step that triggers a Databricks job to run a specific notebook. The results of the job are published back to Azure DevOps.

## Bash Script for Integration Testing

The integration testing step utilizes a Bash script to:

1. Create a job to run the `08-batch-test` notebook.
2. Trigger the job and retrieve the run ID.
3. Wait for the job to complete.
4. Delete the job after completion.
5. Publish the job result.

### Bash Script Content

```bash
#!/bin/bash

# Create a job to run the batch-test notebook
JOB_ID=$(databricks jobs create --json '{
  "name": "Batch Test Job",
  "notebook_task": {
    "notebook_path": "/deployment/08-batch-test"
  },
  "new_cluster": {
    "spark_version": "7.3.x-scala2.12",
    "node_type_id": "Standard_DS3_v2",
    "num_workers": 1
  }
}' | jq -r '.job_id')

# Trigger the job
RUN_ID=$(databricks jobs run-now --job-id $JOB_ID | jq -r '.run_id')

# Wait for the job to complete
while true; do
  STATUS=$(databricks runs get --run-id $RUN_ID | jq -r '.state.life_cycle_state')
  if [[ "$STATUS" == "TERMINATED" ]]; then
    break
  fi
  sleep 5
done

# Delete the job
databricks jobs delete --job-id $JOB_ID

# Publish the job result
if [[ "$STATUS" == "SUCCESS" ]]; then
  echo "Integration test passed."
else
  echo "Integration test failed."
  exit 1
fi
```

## Conclusion

In this project, we successfully created both build and release pipelines in Azure DevOps, configured environment variables for deployment, and implemented an integration testing step to ensure the reliability of our application. 

Feel free to customize and expand this project according to your specific requirements. Happy coding!
