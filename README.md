# AWS HealthOmics Dependencies and Private Workflow Pipeline

# Introduction
AWS HealthOmics is a powerful service that helps researchers, scientists, and bioinformaticians store, query, analyze, and generate insights from genomics and other biological data. It simplifies and accelerates the process of working with genomic information, making scientific discovery and insight generation faster.

HealthOmics has three primary components:
1. **HealthOmics Storage**: Provides efficient and cost-effective storage for petabytes of genomics data.
2. **HealthOmics Analytics**: Simplifies the preparation of genomics data for multiomics and multimodal analyses.
3. **HealthOmics Workflows**: Automatically provisions and scales the underlying infrastructure for bioinformatics computations.

Within HealthOmics Workflows, users can leverage two types of workflows: Ready2Run workflows and private workflows. Private workflows are custom workflows that users can create and define using languages like CWL, WDL, or Nextflow. These private workflows can be shared with other AWS accounts, and users can also accept private workflow shares from other accounts.

However, creating a private workflow in HealthOmics requires a few pre-requisites, including:
1. **Pulling and Pushing Containers to Amazon ECR**: Users must first pull the necessary container images from public or private sources, and then push them to their own Amazon Elastic Container Registry (ECR) repository.
2. **Uploading Workflow Files to Amazon S3**: Users must also upload their workflow definition files (e.g., Nextflow, CWL, or WDL) to an Amazon S3 bucket before they can create the private workflow in HealthOmics.

To streamline this process, the code provided in this repository automates these two pre-requisites, making it easier for users to set up their private workflows in HealthOmics, and allow users to have version controls for each workflow. The code will create two pipelines in AWS CodePipeline:
1. **Workflow Dependencies Pipeline**
   - Pull container images from various sources
   - Scan pulled container images with Trivy and send the reports to the users
2. **Create private workflow in HealthOmics**:
   - Upload workflow definition files to an Amazon S3 bucket
   - Create private workflow on HealthOmics
   - Create private repository in Amazon ECR and push the containers with latest tag and current date (YYYY-MM-DD) as tag

Both pipelines are using CodePipeline V2 version which offers support for monorepo where a single repository hosts multiple projects. This approach consolidates multiple workflows within a single repository, streamlining the development process and reducing the need for maintaining multiple repositories.

# Architecture

## Workflow Dependencies Pipeline

The workflow dependencies pipeline automates the process of pulling the latest container images, scanning them for vulnerabilities, and pushing the approved images to the user's Amazon ECR repository. The architecture for this pipeline is as follows:

![architecture](/architecture-for-workflow-dependencies-pipeline.png)

In this pipeline, the user updates the container-requirement.txt file, which triggers an AWS CodeBuild job to pull the latest container images, scan them using Trivy, and upload the scan results to an Amazon S3 bucket. An AWS Lambda function is then triggered to send the results through email to the user via an SNS. The user can then review the scan results and approve the pipeline, which then the approved containers are pushed to the user's Amazon ECR repository with the 'latest' and timestamp tags. This process will avoid pushing the updates directly to ECR without user acknowledgement.

Once the containers are pushed to the container registry, you can enable enhanced scanning and choose continuous scanning to ensure that the containers are continuously scanned for vulnerabilities and security issues. 

## HealthOmics Private Workflow Pipeline

The HealthOmics private workflow pipeline automates the process of compressing and uploading a new workflow definition file to the user's Amazon S3 bucket, and then creating a new private workflow in AWS HealthOmics. The architecture for this pipeline is as follows:

![architecture](/architecture-for-healthomics-pipeline.png)

## Deployment

> [!IMPORTANT]
> Each deployment of this solution is intended for a single private workflow. If you have multiple private workflows, you must deploy a separate solution for each one.
> By using this software, you agree to the license agreement of the software that is installed.

### Prerequisites
1. A GitHub repository for your workflows.
2. To trigger the Workflow Containers Dependencies Pipeline, you must have a file named `container-requirements.txt` that contains the list of container image URLs required by your HealthOmics private workflow. You may refer to the [sample `container-requirements.txt` file](/sample-workflow/container-requirements.txt).

Follow these steps to implement in your environment:

1. Deploy the `codepipeline.yaml` CloudFormation template. The following CloudFormation parameters are provided:
   - HealthOmicsWorkflowName: The name of the private workflow name for HealthOmics. This will also be identified as the folder name that will trigger the CodePipeline.
   - RepoName: GitHub repo name.
   - RepoBranchName: The GitHub repo branch name.
   - NotificationEmailAddresses: Comma-delimited string of email addresses to subscribe. These emails will receive the scan results and approval notifications.
   - WorkflowFileName: The workflow definition file name.
   - WorkflowParameterFileName: The workflow parameters file name.

> [!IMPORTANT]
> Workflow-related files in the GitHub repository must be contained within a folder named after the workflow name specified in the `HealthOmicsWorkflowName` parameter. For example, if the value is `sample-workflow`, all files must be stored under the `sample-workflow` folder.

2. In order for CodePipeline to access your GitHub repository, the connection to the GitHub repository will need to be approved. Refer to this link - https://docs.aws.amazon.com/dtconsole/latest/userguide/connections-update.html for the steps.


## Clean Up
- Open the [CloudFormation console](https://console.aws.amazon.com/cloudformation).
- Select the stack `codepipeline.yaml` you created then click **Delete**. Wait for the stack to be deleted.
- Open the [ECR console](https://console.aws.amazon.com/ecr).
- Delete the repositories with names starting with the stack name specified during the deployment.
- Open the [HealthOmics console](https://console.aws.amazon.com/omics).
- Delete private workflows based on the workflow name specified during the deployment.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
