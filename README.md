# Deploying microservice applications in AKS using ACR image repositories with Helm, Azure DevOps Infra (bicep) and App Pipelines

Prerequisites

    A valid Azure subscription : https://portal.azure.com/

    Azure CLI : https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest

    kubectl: https://kubernetes.io/docs/tasks/tools/

    bash shell: Most Linux/macOS systems already have this. For Windows, use Git Bash or WSL: https://www.atlassian.com/git/tutorials/git-bash

    Docker Desktop for building and pushing images locally: https://docs.docker.com/desktop/    

    Install Bicep: https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install

    Install Helm: https://helm.sh/docs/intro/install/

# Login to Azure

    az login
    az account set --subscription=<subscriptionId>
    az account show

# Show existing resources

    az resource list

# Create Resources

    ./infra/azcli/script.sh

# Connect to cluster

    RESOURCE_GROUP="rg-onlinestore-dev-uksouth-001"
    AKS_NAME="aks-onlinestore-dev-uksouth-001"
    az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME

# Short name for kubectl

    alias k=kubectl

# Show all existing objects

    k get all


# Log in to ACR

    ACR_NAME="acronlinestoredevuksouth001"
    az acr login --name $ACR_NAME

# Build and push the Docker images to ACR

    # Order Service
    docker build -t order ./app/order-service 
    docker tag order:latest $ACR_NAME.azurecr.io/order:v1
    docker push $ACR_NAME.azurecr.io/order:v1

    # Product Service
    docker build -t product ./app/product-service 
    docker tag product:latest $ACR_NAME.azurecr.io/product:v1
    docker push $ACR_NAME.azurecr.io/product:v1

    # Store Front Service
    docker build -t store-front ./app/store-front
    docker tag store-front:latest $ACR_NAME.azurecr.io/store-front:v1
    docker push $ACR_NAME.azurecr.io/store-front:v1

# helmify 

    helmify -f ./manifests helmchart

# Helm Deploy

    helm install store-release ./helmchart

# Verify the Deployment

    k get pods
    k get services
    Browse the app using http://<LoadBalancer public IP>:80

# Clean the Azure resources

    az group delete --name rg-onlinestore-dev-uksouth-001 --yes --no-wait