# Use Kustomize to deploy Online Boutique with variants
This page walks you through the deployment options for the [Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) sample application on a [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine) cluster using Kustomize. Deployment variants are designed as **components**, meaning that multiple variant components can be enabled at once.

## What is Kustomize?
Kustomize is a Kubernetes native configuration management tool that allows users to customize their manifest configurations without having to copy entire files multiple times. Its commands are built into `kubectl` as `apply -k`. This repo uses Kustomize to enable the deployment variants found [here](https://github.com/GoogleCloudPlatform/microservices-demo#other-deployment-options). More information can be found on the official [Kustomize website](https://kustomize.io/).

## Supported Deployment Variations
| **Variation Name**                                                                                                         | **Description**                                                                                                                                                                                                                                                                                |
|----------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Cymbal Branding](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/docs/cymbal-shops.md)                | Changes all Online Boutique-related branding to Google Cloud's fictitious company--Cymbal Shops. The code adds/enables an environment variable `CYMBAL_BRANDING` to the `frontend` deployment.                                                                                                 |
| [Google Cloud Operations](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/docs/gcp-instrumentation.md) | Enables Monitoring (Stats), Tracing, Profiler, and Debugger for various services within Online Boutique. The code removes the existing environment variables (`DISABLE_STATS`, `DISABLE_TRACING`, `DISABLE_PROFILER`, `DISABLE_DEBUGGER`) from appropriate YAML config files.                  |
| [Memorystore](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/docs/memorystore.md)                     | The vanilla Online Boutique deployment uses the default in-cluster `redis` database for storing the contents of its shopping cart. The Memorystore deployment variation overrides the default database with its own Memorystore (redis) database. These changes directly affect `cartservice`. |

## Deployment Instructions (Memorystore)
1. While in the `microservices-demo` directory, enter the `terraform/` directory.
    ```
    cd terraform
    ```

1. Open the `terraform.tfvars` file. Replace `<project_id_here>` with the [GCP Project ID](https://cloud.google.com/resource-manager/docs/creating-managing-projects?hl=en#identifying_projects) for the `gcp_project_id` variable. Change the value of `memorystore = false` to `memorystore = true`.

1. Initialize Terraform.
    ```
    terraform init
    ```

1. See what resources will be created.
    ```
    terraform plan
    ```

1. Create the resources and deploy the updated infrastructure.
    ```
    terraform apply
    ```
    1. If there is a confirmation prompt, type `yes` and hit Enter/Return.
    
    Note: Following completion, there may be an error saying that the cluster already exists-- this is okay.

1. While in the `terraform/` directory, enter the `kustomize/` directory.
    ```
    cd ../kustomize
    ```

1. Edit the base level `kustomization.yaml` so that it is using the Memorystore component.
    ```
    vim kustomization.yaml
    ```
    The code file should contain the following snippet of code.
    ```
    components:
      - components/memorystore
    ```

1. Check to see what changes will be made to the existing deployment config.
    ```
    kustomize build .
    ```

1. Apply the Kustomize deployment changes to the existing deployment.
    ```
    kubectl apply -k .
    ```
    Note: It may take 2-3 minutes before the changes are reflected on the deployment.

### Cleanup (Memorystore)
After you have run the deployment variant on Online Boutique, you will want to reset the sample application back to its vanilla state.

1. While still in the `kustomize/` directory, re-apply the original Kubernetes config to the Online Boutique deployment.
    ```
    kubectl apply -f base
    ```

1. Enter the `terraform/` directory.
    ```
    cd ../terraform
    ```

1. Undo the additional Terraform infrastructure by targeting the Memorystore instance.
    ```
    terraform destroy -target=google_redis_instance.redis-cart
    ```
    1. If there is a confirmation prompt, type `yes` and hit Enter/Return.

## Deployment Instructions (General)
1. While in the `microservices-demo` directory, enter the `kustomize/` directory.
    ```
    cd kustomize
    ```

1. Edit the base level `kustomization.yaml` so that it is uses the intended components.
    ```
    vim kustomization.yaml
    ```

    The file should contain a similar snippet of code with only the component variants that you intend to deploy.
    ```
    components:
      - components/cymbal-branding
      - components/google-cloud-operations
      # - components/memorystore
     ```

    Note: The example above will enable both the Cymbal Branding and Google Cloud Operations components in the new deployment.

1. Check to see what changes will be made to the existing deployment config.
    ```
    kustomize build .
    ```

1. Apply the Kustomize deployment changes to the existing deployment.
    ```
    kubectl apply -k .
    ```

    Note: It may take 2-3 minutes before the changes are reflected on the deployment.

### Cleanup (General)
After you have run the deployment variant on Online Boutique, you will want to reset the sample application back to its vanilla state.

1. While still in the `kustomize/` directory, re-apply the original Kubernetes config to the Online Boutique deployment.
    ```
    kubectl apply -f base
    ```
    
    Note: It may take 2-3 minutes before the changes are reflected on the deployment.
    