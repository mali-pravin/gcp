---
sidebar_position: 1
hide_table_of_contents: true
title: Google Cloud Function
---

# Google Cloud Function Deployment

```mdx-code-block
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
```

This tutorial will get you started with Harness Continuous Delivery (CD). We will guide you through deploying new Cloud Functions to Google Cloud using Harness CD pipeline. 

:::info

[Sign up today to unleash the potential of intelligent Harness CD](https://app.harness.io/auth/#/signup/?module=cd&utm_source=website&utm_medium=harness-developer-hub&utm_campaign=cd-plg&utm_content=tutorials-cd-kubernetes-gcp-function).

:::

Harness CD pipelines allow you to orchestrate and automate your deployment workflows, and push updated application images to your target Kubernetes cluster. Pipelines allow extensive control over how you want to progress artifacts through various dev / test / stage / prod clusters, while running a variety of scans & tests to ensure quality and stability standards you and team may have defined.
## Before you begin

Verify that you have the following:

1. **Obtain GitHub personal access token with the repo scope**. See the GitHub documentation on [creating a personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line).
2. **A Kubernetes cluster**. Use your own Kubernetes cluster or we recommend using [K3D](https://k3d.io/v5.5.1/) for installing Harness Delegates and deploying a sample application in a local development environment.
    - Check [Delegate system requirements](https://developer.harness.io/docs/platform/Delegates/delegate-concepts/delegate-requirements).
3. **Install the [Docker](https://helm.sh/docs/intro/install/)** in order to install the Docker delegate.
4. **Fork the [harnessed-example-apps](https://github.com/harness-community/harnesscd-example-apps/fork)** repository through the GitHub website.
    - For details on Forking a GitHub repository, go to [GitHub docs](https://docs.github.com/en/get-started/quickstart/fork-a-repo#forking-a-repository) for more information on forking a GitHub repository.
5. ** Google Cloud Function and Google Cloud Storage Permission**. Harness supports Google Cloud Functions 1st and 2nd gen. There are minor differences in the permissions required by each generation. For a detailed breakdown, go to [Access control with IAM from Google](https://cloud.google.com/functions/docs/concepts/iam).


## Google Cloud Function Support 

Cloud Functions offers two product versions: Cloud Functions (1st gen), the original version, and Cloud Functions (2nd gen), a new version built on Cloud Run and Eventarc to provide an enhanced feature set. Harness supports deploying Google Functions 1st gen and 2nd gen.

To get more information on Google Function 1st gen and 2nd gen, refer [Cloud Functions version comparison](https://cloud.google.com/functions/docs/concepts/version-comparison).

We suggest opting for Cloud Functions (2nd gen) whenever feasible for new functions, as it offers a range of additional features. You can refer [New in Cloud Functions(2nd Gen)](https://cloud.google.com/functions/docs/concepts/version-comparison#new-in-2nd-gen) doc for more information.


## Cloud Functions Limitations
- For Google Cloud Functions 2nd gen, Harness does not support [Google Cloud Source Repository](https://cloud.google.com/functions/docs/deploy#from-source-repo) at this time. Only Google Cloud Storage is supported.
- For Google Cloud Functions 1st gen, Harness supports both Google Cloud Storage and Google Cloud Source.

## Steps to perform on Google Cloud 

- Make sure you have appropriate permission on the **Google Cloud Function** and **Google Cloud Storage** as mentioned above.
- Download the **hello_world.zip** for 1st gen and **helloHttp.zip** for 2nd gen files from [GitHub](https://github.com/harnesspravin/harnesscd-example-apps/tree/master/google_cloud_function).
- Login to Google Cloud console and create a Bucket, Refer [Create a new bucket](https://cloud.google.com/storage/docs/creating-buckets) doc from Google Cloud.
- Once the bucket is created, Click on the UPLOAD FILES and upload **hello_world.zip** and **helloHttp.zip** files.

```mdx-code-block
<Tabs>
<TabItem value="2nd Gen">
```

## Getting Started with Harness CD
----------------------------------

1. Login to [Harness](https://app.harness.io/).

2. Select **Projects**, and then select **Default Project**

:::caution

For the pipeline to run successfully, please follow the remaining steps as they are, including the naming conventions.

:::

### Delegate

<details open>
<summary>What is the Harness delegate?</summary>

The Harness delegate is a service that runs in your local network or VPC to establish connections between the Harness Manager and various providers such as artifacts registries, cloud platforms, etc. The delegate is installed in the target infrastructure, for example, a Kubernetes cluster, and performs operations including deployment and integration. Learn more about the delegate in the [Delegate Overview](https://developer.harness.io/docs/platform/delegates/delegate-concepts/delegate-overview/).

</details>

3. Under **Project Setup**, select **Delegates**.
    - Select **Delegates**.
        - Select **New Delegate**.
        - Click on Docker
          
          For this tutorial, let's explore how to install a delegate using Docker.

        -  In the command provided, `ACCOUNT_ID` and `MANAGER_ENDPOINT` are auto-populated values that you can obtain from the delegate installation wizard.
        -  Replace **DELEGATE_TOKEN** in the command with the token that was copied earlier and proceed with delegate installation.

         
        ```bash
        docker run -dit --cpus=1 --memory=2g \
         -e DELEGATE_NAME=docker-delegate \
         -e NEXT_GEN="true" \
         -e DELEGATE_TYPE="DOCKER" \
         -e ACCOUNT_ID=ACCOUNT_ID \
         -e DELEGATE_TOKEN=DELEGATE_TOKEN \
         -e LOG_STREAMING_SERVICE_URL=https://app.harness.io/gratis/log-service/ \
         -e MANAGER_HOST_AND_PORT=https://app.harness.io/gratis harness/delegate:23.05.79310
        ```
        - Select **Verify** to verify that the delegate is installed successfully and can connect to the Harness Manager.

:::note

You can also follow the [Install Harness Delegate on Kubernetes or Docker](https://developer.harness.io/tutorials/platform/install-delegate/) tutorial to install the delegate using the Harness Terraform Provider or a Kubernetes manifest.

:::


### Secrets

<details open>
<summary>What are Harness secrets?</summary>

Harness offers built-in secret management for encrypted storage of sensitive information. Secrets are decrypted when needed, and only the private network-connected Harness delegate has access to the key management system. You can also integrate your own secret manager. To learn more about secrets in Harness, go to [Harness Secret Manager Overview](https://developer.harness.io/docs/platform/Secrets/Secrets-Management/harness-secret-manager-overview/).

</details>

4. Under **Project Setup**, select **Secrets**.
    - Select **New Secret**, and then select **Text**.
    - Enter the secret name `harness_gitpat`.
    - For the secret value, paste the GitHub personal access token you saved earlier.
    - Select **Save**.

5. Under **Project Setup**, select **Secrets**.
    - Select **New Secret**, and then select **File**.
    - Enter the secret name `gcp`.
    - For the secret value, Upload the credentials for your service account which you can download from [gcp project](https://developers.google.com/workspace/guides/create-credentials#create_credentials_for_a_service_account).
    - Select **Save**.

### Connectors

<details open>
<summary>What are connectors?</summary>

Connectors in Harness enable integration with 3rd party tools, providing authentication and operations during pipeline runtime. For instance, a GitHub connector facilitates authentication and fetching files from a GitHub repository within pipeline stages. Explore connector how-tos [here](https://developer.harness.io/docs/category/connectors).

</details>

5. Create the **GitHub connector**.
    - Copy the contents of [1-github-connector.yml](https://github.com/harness-community/harnesscd-example-apps/blob/master/guestbook/harnesscd-pipeline/1-github-connector.yml)
    - In your Harness project in the Harness Manager, under **Project Setup**, select **Connectors**.
    - Select **Create via YAML Builder** and paste the copied YAML.
    - Assuming you have already forked the [harnessed-example-apps](https://github.com/harness-community/harnesscd-example-apps/fork) repository mentioned earlier, replace **GITHUB_USERNAME** with your GitHub account username in the YAML.
    - In `projectIdentifier`, verify that the project identifier is correct. You can see the Id in the browser URL (after `account`). If it is incorrect, the Harness YAML editor will suggest the correct Id.
    - Select **Save Changes** and verify that the new connector named **harness_gitconnector** is successfully created.
    - Finally, select **Connection Test** under **Connectivity Status** to ensure the connection is successful.

6. Create the **GCP connector**.
    - Copy the contents of [2-gcp-connector.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/2-gcp-connector.yml).
    - In your Harness project, under **Project Setup**, select **Connectors**.
    - Select **Create via YAML Builder** and and paste the copied YAML.
    - Select **Save Changes** and verify that the new connector named **gcpconnector** is successfully created.
    - Finally, select **Connection Test** under **Connectivity Status** to verify the connection is successful.

### Environment

<details open>
<summary>What are Harness environments?</summary>

Environments define the deployment location, categorized as **Production** or **Pre-Production**. Each environment includes infrastructure definitions for VMs, Kubernetes clusters, or other target infrastructures. To learn more about environments, go to [Environments overview](https://developer.harness.io/docs/continuous-delivery/x-platform-cd-features/environments/environment-overview/).

</details>

7. In your Harness project, select **Environments**.
    - Select **New Environment**, and then select **YAML**.
    - Copy the contents of [3-environment.yml](https://github.com/harness-community/harnesscd-example-apps/blob/master/guestbook/harnesscd-pipeline/3-environment.yml), paste it into the YAML editor, and select **Save**.
    - In your new environment, select the **Infrastructure Definitions** tab.
    - Select **Infrastructure Definition**, and then select **YAML**.
    - Copy the contents of [4-infrastructure-definition.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/4-infrastructure-definition.yml) and paste it into the YAML editor.
    - Replace `GCP PROJECT NAME` with your gcp project name.
    - Replace `GCP REGION` with your gcp region name.
    - Select **Save** and verify that the environment and infrastructure definition are created successfully.

### Services

<details open>
<summary>What are Harness services?</summary>

In Harness, services represent what you deploy to environments. You use services to configure variables, manifests, and artifacts. The **Services** dashboard provides service statistics like deployment frequency and failure rate. To learn more about services, go to [Services overview](https://developer.harness.io/docs/continuous-delivery/x-platform-cd-features/services/services-overview/).

</details>


8. In your Harness project, select **Services**.
    - Select **New Service**.
    - Enter the name `hello_http`.
    - Select **Save**, and then **YAML** (on the **Configuration** tab).
    - Select **Edit YAML**, copy the contents of [5-service-gen2.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/5-service-gen2.yml), and paste the into the YAML editor.
    - Replace `GCP PROJECT` with your gcp project name.
    - Replace `GCP BUCKET` with your gcp bucket name.
    - Select **Save**, and verify that the service **hello_http** is successfully created.

### Pipeline

<details open>
<summary>What are Harness pipelines?</summary>

A pipeline is a comprehensive process encompassing integration, delivery, operations, testing, deployment, and monitoring. It can utilize CI for code building and testing, followed by CD for artifact deployment in production. A CD Pipeline is a series of stages where each stage deploys a service to an environment. To learn more about CD pipeline basics, go to [CD pipeline basics](https://developer.harness.io/docs/continuous-delivery/get-started/cd-pipeline-basics/).

</details>

9. In your Harness project, select **Pipelines**.
- Select **Create a Pipeline**.
- Enter the name `hello_http`.
- Select **Inline** to store the pipeline in Harness.
- Select **Start**. 
- In **Pipeline Studio**, select **YAML**.
- Select **Edit YAML** and choose any of the following execution strategies. Paste the respective YAML based on your selection.

```mdx-code-block
<Tabs>
<TabItem value="Canary">
```

<details open>
<summary>What are Canary deployments?</summary>

A canary deployment updates nodes in a single environment gradually, allowing you to use gates between increments. Canary deployments allow incremental updates and ensure a controlled rollout process. For more information, go to [When to use Canary deployments](https://developer.harness.io/docs/continuous-delivery/manage-deployments/deployment-concepts#when-to-use-canary-deployments).

</details>

- Copy the contents of [6-canary-deployment-gen2.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/6-canary-deployment-gen2.yml).

- In your Harness pipeline YAML editor, paste the YAML.

- Select **Save**.
   
   You can switch to the **Visual** editor and confirm the pipeline stage and execution steps as shown below.

<docimage path={require('../static/harness-cicd-tutorial/canary-gen2.png')} width="60%" height="60%" title="Click to view full size image" />


```mdx-code-block
</TabItem>
<TabItem value="Blue Green">
```

<details open>
<summary>What are Blue Green deployments?</summary>

Blue Green deployments involve running two identical environments (stage and prod) simultaneously with different service versions. QA and UAT are performed on a **new** service version in the stage environment first. Next, traffic is shifted from the prod environment to stage, and the previous service version running on prod is scaled down. Blue Green deployments are also referred to as red/black deployment by some vendors. For more information, go to [When to use Blue Green deployments](https://developer.harness.io/docs/continuous-delivery/manage-deployments/deployment-concepts#when-to-use-blue-green-deployments).

</details>


- Copy the contents of [6-bluegreen-deployment-gen2.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/6-bluegreen-deployment-gen2.yml).

- In your Harness pipeline YAML editor, paste the YAML.

- Select **Save**.
   
   You can switch to the **Visual** pipeline editor and confirm the pipeline stage and execution steps as shown below.

<docimage path={require('../static/harness-cicd-tutorial/bluegreen-gen2.png')} width="60%" height="60%" title="Click to view full size image" />


```mdx-code-block
</TabItem>
<TabItem value="Rolling">
```

<details open>
<summary>What are Rolling deployments?</summary>

Rolling deployments incrementally add nodes in a single environment with a new service version, either one-by-one or in batches defined by a window size. Rolling deployments allow a controlled and gradual update process for the new service version. For more information, go to [When to use rolling deployments](https://developer.harness.io/docs/continuous-delivery/manage-deployments/deployment-concepts#when-to-use-rolling-deployments).

</details>



- Copy the contents of [6-rolling-deployment-gen2.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/6-rolling-deployment-gen2.yml).

- In your Harness pipeline YAML editor, paste the YAML.

- Select **Save**.
   
   You can switch to the **Visual** pipeline editor and confirm the pipeline stage and execution steps as shown below.

<docimage path={require('../static/harness-cicd-tutorial/rolling-gen2.png')} width="60%" height="60%" title="Click to view full size image" />


```mdx-code-block
</TabItem>
</Tabs>
```

Finally, it's time to execute your pipeline. 

10.  Select **Run**, and then select **Run Pipeline** to initiate the deployment.
  - Observe the execution logs as Harness deploys the workload and checks for steady state.
  - After a successful execution, you can check the deployment on your Google Cloud Function console.
    
### Congratulations!🎉

You've just learned how to use Harness CD to deploy an application using a Google Cloud Functions gen 2 to Google Cloud.

#### What's Next?

- Keep learning about Harness CD. For example, add [Triggers](https://developer.harness.io/docs/platform/Triggers/triggering-pipelines) to your pipeline that initiate pipeline deployments in response to Git events.
- Visit the [Harness Developer Hub](https://developer.harness.io/) for more tutorials and resources.

```mdx-code-block
</TabItem>
<TabItem value="1st Gen">
```
## Getting Started with Harness CD
----------------------------------

1. Login to [Harness](https://app.harness.io/).

2. Select **Projects**, and then select **Default Project**

:::caution

For the pipeline to run successfully, please follow the remaining steps as they are, including the naming conventions.

:::

### Delegate

<details open>
<summary>What is the Harness delegate?</summary>

The Harness delegate is a service that runs in your local network or VPC to establish connections between the Harness Manager and various providers such as artifacts registries, cloud platforms, etc. The delegate is installed in the target infrastructure, for example, a Kubernetes cluster, and performs operations including deployment and integration. Learn more about the delegate in the [Delegate Overview](https://developer.harness.io/docs/platform/delegates/delegate-concepts/delegate-overview/).

</details>

3. Under **Project Setup**, select **Delegates**.
    - Select **Delegates**.
        - Select **New Delegate**.
          
          For this tutorial, let's explore how to install a delegate using Helm.

        - Add the Harness Helm chart repo to your local helm registry using the following commands.
        
        ```bash
        helm repo add harness-delegate https://app.harness.io/storage/harness-download/delegate-helm-chart/
        ```

        - Update the repo:

        ```bash
        helm repo update harness-delegate
        ```

        -  In the command provided, `ACCOUNT_ID` and `MANAGER_ENDPOINT` are auto-populated values that you can obtain from the delegate installation wizard.
        -  Replace **DELEGATE_TOKEN** in the command with the token that was copied earlier and proceed with delegate installation.

         
        ```bash
        helm upgrade -i helm-delegate --namespace harness-delegate-ng --create-namespace \
        harness-delegate/harness-delegate-ng \
         --set delegateName=helm-delegate \
         --set accountId=ACCOUNT_ID \
         --set managerEndpoint=MANAGER_ENDPOINT \
         --set delegateDockerImage=harness/delegate:23.03.78904 \
         --set replicas=1 --set upgrader.enabled=false \
         --set delegateToken=DELEGATE_TOKEN
        ```
        - Select **Verify** to verify that the delegate is installed successfully and can connect to the Harness Manager.

:::note

You can also follow the [Install Harness Delegate on Kubernetes or Docker](https://developer.harness.io/tutorials/platform/install-delegate/) tutorial to install the delegate using the Harness Terraform Provider or a Kubernetes manifest.

:::


### Secrets

<details open>
<summary>What are Harness secrets?</summary>

Harness offers built-in secret management for encrypted storage of sensitive information. Secrets are decrypted when needed, and only the private network-connected Harness delegate has access to the key management system. You can also integrate your own secret manager. To learn more about secrets in Harness, go to [Harness Secret Manager Overview](https://developer.harness.io/docs/platform/Secrets/Secrets-Management/harness-secret-manager-overview/).

</details>

4. Under **Project Setup**, select **Secrets**.
    - Select **New Secret**, and then select **Text**.
    - Enter the secret name `harness_gitpat`.
    - For the secret value, paste the GitHub personal access token you saved earlier.
    - Select **Save**.

### Connectors

<details open>
<summary>What are connectors?</summary>

Connectors in Harness enable integration with 3rd party tools, providing authentication and operations during pipeline runtime. For instance, a GitHub connector facilitates authentication and fetching files from a GitHub repository within pipeline stages. Explore connector how-tos [here](https://developer.harness.io/docs/category/connectors).

</details>

5. Create the **GitHub connector**.
    - Copy the contents of [1-github-connector.yml](https://github.com/harness-community/harnesscd-example-apps/blob/master/guestbook/harnesscd-pipeline/1-github-connector.yml)
    - In your Harness project in the Harness Manager, under **Project Setup**, select **Connectors**.
    - Select **Create via YAML Builder** and paste the copied YAML.
    - Assuming you have already forked the [harnessed-example-apps](https://github.com/harness-community/harnesscd-example-apps/fork) repository mentioned earlier, replace **GITHUB_USERNAME** with your GitHub account username in the YAML.
    - In `projectIdentifier`, verify that the project identifier is correct. You can see the Id in the browser URL (after `account`). If it is incorrect, the Harness YAML editor will suggest the correct Id.
    - Select **Save Changes** and verify that the new connector named **harness_gitconnector** is successfully created.
    - Finally, select **Connection Test** under **Connectivity Status** to ensure the connection is successful.

6. Create the **Kubernetes connector**.
    - Copy the contents of [2-kubernetes-connector.yml](https://github.com/harness-community/harnesscd-example-apps/blob/master/guestbook/harnesscd-pipeline/2-kubernetes-connector.yml).
    - In your Harness project, under **Project Setup**, select **Connectors**.
    - Select **Create via YAML Builder** and and paste the copied YAML.
    - Replace **DELEGATE_NAME** with the installed Delegate name. To obtain the Delegate name, navigate to **Project Setup**, and then **Delegates**.
    - Select **Save Changes** and verify that the new connector named **harness_k8sconnector** is successfully created.
    - Finally, select **Connection Test** under **Connectivity Status** to verify the connection is successful.

### Environment

<details open>
<summary>What are Harness environments?</summary>

Environments define the deployment location, categorized as **Production** or **Pre-Production**. Each environment includes infrastructure definitions for VMs, Kubernetes clusters, or other target infrastructures. To learn more about environments, go to [Environments overview](https://developer.harness.io/docs/continuous-delivery/x-platform-cd-features/environments/environment-overview/).

</details>

7. In your Harness project, select **Environments**.
    - Select **New Environment**, and then select **YAML**.
    - Copy the contents of [3-environment.yml](https://github.com/harness-community/harnesscd-example-apps/blob/master/guestbook/harnesscd-pipeline/3-environment.yml), paste it into the YAML editor, and select **Save**.
    - In your new environment, select the **Infrastructure Definitions** tab.
    - Select **Infrastructure Definition**, and then select **YAML**.
    - Copy the contents of [4-infrastructure-definition.yml](https://github.com/harness-community/harnesscd-example-apps/blob/master/guestbook/harnesscd-pipeline/4-infrastructure-definition.yml) and paste it into the YAML editor.
    - Select **Save** and verify that the environment and infrastructure definition are created successfully.

### Services

<details open>
<summary>What are Harness services?</summary>

In Harness, services represent what you deploy to environments. You use services to configure variables, manifests, and artifacts. The **Services** dashboard provides service statistics like deployment frequency and failure rate. To learn more about services, go to [Services overview](https://developer.harness.io/docs/continuous-delivery/x-platform-cd-features/services/services-overview/).

</details>


8. In your Harness project, select **Services**.
    - Select **New Service**.
    - Enter the name `hello_world`.
    - Select **Save**, and then **YAML** (on the **Configuration** tab).
    - Select **Edit YAML**, copy the contents of [7-service-gen1.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/7-service-gen1.yml) , and paste the into the YAML editor.
    - Replace `project` with your gcp project name.
    - Replace `bucket` with your gcp bucket name.
    - Replace `artifactPath` with the `hello_world.zip` file path from google bucket 
    - Select **Save**, and verify that the service **hello_world** is successfully created.

### Pipeline

<details open>
<summary>What are Harness pipelines?</summary>

A pipeline is a comprehensive process encompassing integration, delivery, operations, testing, deployment, and monitoring. It can utilize CI for code building and testing, followed by CD for artifact deployment in production. A CD Pipeline is a series of stages where each stage deploys a service to an environment. To learn more about CD pipeline basics, go to [CD pipeline basics](https://developer.harness.io/docs/continuous-delivery/get-started/cd-pipeline-basics/).

</details>

9. In your Harness project, select **Pipelines**.
- Select **Create a Pipeline**.
- Enter the name `hello_world`.
- Select **Inline** to store the pipeline in Harness.
- Select **Start**. 
- In **Pipeline Studio**, select **YAML**.
- Select **Edit YAML** and choose any of the following execution strategies. Paste the respective YAML based on your selection.

```mdx-code-block
<Tabs>
<TabItem value="Canary">
```

<details open>
<summary>What are Canary deployments?</summary>

A canary deployment updates nodes in a single environment gradually, allowing you to use gates between increments. Canary deployments allow incremental updates and ensure a controlled rollout process. For more information, go to [When to use Canary deployments](https://developer.harness.io/docs/continuous-delivery/manage-deployments/deployment-concepts#when-to-use-canary-deployments).

</details>

- Copy the contents of [7-canary-deployment-gen1.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/7-canary-deployment-gen1.yml).

- In your Harness pipeline YAML editor, paste the YAML.

- Select **Save**.
   
   You can switch to the **Visual** editor and confirm the pipeline stage and execution steps as shown below.

<docimage path={require('../static/harness-cicd-tutorial/canary-gen2.png')} width="60%" height="60%" title="Click to view full size image" />

```mdx-code-block
</TabItem>
<TabItem value="Blue Green">
```

<details open>
<summary>What are Blue Green deployments?</summary>

Blue Green deployments involve running two identical environments (stage and prod) simultaneously with different service versions. QA and UAT are performed on a **new** service version in the stage environment first. Next, traffic is shifted from the prod environment to stage, and the previous service version running on prod is scaled down. Blue Green deployments are also referred to as red/black deployment by some vendors. For more information, go to [When to use Blue Green deployments](https://developer.harness.io/docs/continuous-delivery/manage-deployments/deployment-concepts#when-to-use-blue-green-deployments).

</details>


- Copy the contents of [7-bluegreen-deployment-gen1.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/7-bluegreen-deployment-gen1.yml).

- In your Harness pipeline YAML editor, paste the YAML.

- Select **Save**.
   
   You can switch to the **Visual** pipeline editor and confirm the pipeline stage and execution steps as shown below.

<docimage path={require('../static/harness-cicd-tutorial/canary-gen2.png')} width="60%" height="60%" title="Click to view full size image" />


```mdx-code-block
</TabItem>
<TabItem value="Rolling">
```

<details open>
<summary>What are Rolling deployments?</summary>

Rolling deployments incrementally add nodes in a single environment with a new service version, either one-by-one or in batches defined by a window size. Rolling deployments allow a controlled and gradual update process for the new service version. For more information, go to [When to use rolling deployments](https://developer.harness.io/docs/continuous-delivery/manage-deployments/deployment-concepts#when-to-use-rolling-deployments).

</details>



- Copy the contents of [7-rolling-deployment-gen1.yml](https://github.com/harnesspravin/harnesscd-example-apps/blob/master/google_cloud_function/7-rolling-deployment-gen1.yml).

- In your Harness pipeline YAML editor, paste the YAML.

- Select **Save**.
   
   You can switch to the **Visual** pipeline editor and confirm the pipeline stage and execution steps as shown below.

<docimage path={require('../static/harness-cicd-tutorial/canary-gen2.png')} width="60%" height="60%" title="Click to view full size image" />

```mdx-code-block
</TabItem>
</Tabs>
```

Finally, it's time to execute your pipeline. 

10.  Select **Run**, and then select **Run Pipeline** to initiate the deployment.
  - Observe the execution logs as Harness deploys the workload and checks for steady state.
  - After a successful execution, you can check the deployment on your Google Cloud Function console.
    
### Congratulations!🎉

You've just learned how to use Harness CD to deploy an application using a Google Cloud Functions gen 1 to Google Cloud.

#### What's Next?
- Keep learning about Harness GitOps. Create a GitOps ApplicationSet and PR Pipeline in Harness GitOps by following this [guide](https://developer.harness.io/docs/continuous-delivery/gitops/harness-git-ops-application-set-tutorial).
- Visit the [Harness Developer Hub](https://developer.harness.io/) for more tutorials and resources.  

```mdx-code-block
</TabItem>
</Tabs>
```
