# Azure DevOps Build and Release Pipeline

## Overview

This project demonstrates how to create a CI/CD pipeline using Azure DevOps to automate the build and release process for a Databricks project. The pipeline includes a build pipeline that packages code artifacts and a release pipeline that deploys the artifacts to a Databricks workspace and runs integration tests.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Creating the Build Pipeline](#creating-the-build-pipeline)
  - [Creating a YAML File](#creating-a-yaml-file)
  - [Committing the YAML File](#committing-the-yaml-file)
  - [Setting Up the Build Pipeline](#setting-up-the-build-pipeline)
- [Creating the Release Pipeline](#creating-the-release-pipeline)
  - [Defining Artifacts and Stages](#defining-artifacts-and-stages)
  - [Configuring Tasks](#configuring-tasks)
- [Integration Testing](#integration-testing)
- [Environment Variables](#environment-variables)
- [Usage](#usage)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

- An Azure DevOps account
- A project set up in Azure DevOps
- Access to a Databricks workspace
- Basic understanding of YAML and Azure DevOps pipelines

## Project Structure

```
.
├── .gitignore
├── build-pipeline.yaml        # YAML file for the build pipeline
├── release-pipeline.yaml      # YAML file for the release pipeline
└── integration-test.sh        # Bash script for integration testing
```

## Creating the Build Pipeline

### Creating a YAML File

1. Log in to your Azure DevOps portal.
2. Navigate to your project (e.g., SBIT).
3. Go to **Repos**.
4. Create a new branch (e.g., `feature-build`).
5. Create a new YAML file (e.g., `build-pipeline.yaml`) and paste the following content:

```yaml
# Example YAML for a simple build pipeline
trigger:
  batch: true
  branches:
    include:
      - '*'

pool:
  vmImage: 'ubuntu-22.04'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.10'
  
- script: |
    pip install pytest requests setuptools wheel
  displayName: 'Install Python Packages'

- script: |
    cp -R $(Build.SourcesDirectory) $(Build.ArtifactStagingDirectory)/binaries
  displayName: 'Copy Latest Code'

- task: ArchiveFiles@2
  inputs:
    rootFolderOrFile: '$(Build.ArtifactStagingDirectory)/binaries'
    includeRootFolder: false
    archiveFile: '$(Build.ArtifactStagingDirectory)/output.zip'
```

### Committing the YAML File

1. Save and commit the YAML file to the `feature-build` branch.
2. Create a pull request to merge into the `release` branch.

### Setting Up the Build Pipeline

1. Go to **Pipelines** in Azure DevOps.
2. Click on **New Pipeline** and select **Azure Repos**.
3. Choose the existing YAML file option and select your `release` branch and `build-pipeline.yaml`.
4. Save the pipeline.

## Creating the Release Pipeline

### Defining Artifacts and Stages

1. Go to **Pipelines** > **Releases** in Azure DevOps.
2. Click **New Release Pipeline** and select an **Empty Job**.
3. In the **Artifacts** section, add the build pipeline as the source.

### Configuring Tasks

1. Define an agent job with `ubuntu-22.04`.
2. Add the following tasks:
   - Use Python Version
   - Extract Files
   - Install Databricks CLI (using a Bash task)
   - Deploy code to Databricks (using another Bash task)
   - Run Integration Tests (using a final Bash task)

## Integration Testing

The integration testing script is defined in `integration-test.sh`. It performs the following steps:

1. Create a job to run the `08-batch-test` notebook.
2. Trigger the job and get the run ID.
3. Wait for the job to complete.
4. Delete the job after completion.
5. Publish the job result.

### Sample Bash Script

```bash
#!/bin/bash

# Create a job to run the integration test
job_id=$(databricks jobs create --json '{
  "name": "Integration Test Job",
  "new_cluster": {
    "num_workers": 1,
    "spark_version": "8.3.x-scala2.12",
    "node_type_id": "i3.xlarge"
  },
  "notebook_task": {
    "notebook_path": "/Deployment/08-batch-test"
  }
}' | jq -r '.job_id')

# Run the job
run_id=$(databricks jobs run-now --job-id "$job_id" | jq -r '.run_id')

# Poll for job completion
while true; do
  status=$(databricks runs get --run-id "$run_id" | jq -r '.state.life_cycle_state')
  if [[ "$status" == "TERMINATED" ]]; then
    break
  fi
  sleep 10
done

# Cleanup
databricks jobs delete --job-id "$job_id"

# Publish results
echo "Integration test completed."
```

## Environment Variables

Make sure to define the following environment variables in the release pipeline:

- `DATABRICKS_HOST`: The URL of your Databricks workspace.
- `DATABRICKS_TOKEN`: The access token for authentication.

## Usage

1. Create a pull request to merge your changes into the `release` branch.
2. After the build pipeline completes, manually trigger the release pipeline from the Azure DevOps portal.
3. Monitor the pipeline runs for successful deployment and integration test results.

## Contributing

If you'd like to contribute to this project, please fork the repository and submit a pull request. All contributions are welcome!

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

Feel free to customize this README according to your project specifics and any additional instructions you'd like to provide!
