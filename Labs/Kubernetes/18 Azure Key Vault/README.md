<img src="../../../img/logo.png" alt="Chmurowisko logo" width="200" align="right">
<br><br>
<br><br>
<br><br>

# AKS and Azure Key Vault

## LAB Overview

#### In this lab you are going to integrate your K8s cluster with Azure KeyVault. Azure Key Vault is a service used to store sensitive data, eg passwords, secrets and certificates.

There is no out-of-the box integration betwenn AKS and Azure Key Vault but you can use FlexVolume project to achieve it. More info on: https://github.com/Azure/kubernetes-keyvault-flexvol


## Task 1: Create Azure Key Vault
First we need to create Azure Key Vault service and store some sensitive information in it.

1. Login to Azure portal: `https://portal.azure.com`
2. Click **Create a resource** button
3. Search for **Key Vault** service, click **Create**
4. Fill the form:
   - **Resource group:** use your own resource group
   - **Key vault name:** choose your name
   - **Region:** West Europe
   - **Pricing Tier:** Standard
5. Go **Next** until last blade. Leave all fields by default.
6. Click **Create** and wait for the deployment to finish

## Task 2: Create custom secret in yout key vault
1. Open your Azure Key Vault service.
2. From the left pane select "Secrets".
3. Click "Generate/Import" button
4. Fill the form:
   - **Upload options:** Manual
   - **Name:** choose your secret's name
   - **Value:** enter your secret's custom value
   - leave all fields by default
5. Click **Create**
6. You should see your new secret on secrets list.

## Task 3: Install FlexVolume
Now we are going install and configure FlexVolume. We will use **Pod Identity** mode to access Azure Key Vault.
1. Install FlexVolume:
   
`kubectl create -f https://raw.githubusercontent.com/Azure/kubernetes-keyvault-flexvol/master/deployment/kv-flexvol-installer.yaml`

It is a daemon set that is installed on every node in a cluster. Pods are created in a new namespace. Check if pods are installed:
`kubectl get pods -n kv`

2. Install AAD pod identity:
   
`kubectl apply -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment-rbac.yaml`

After you execute `kubectl get pods` you will see mic and nmi pods created.

3. Create Azure Managed Identity:
   
`az identity create -g <-resourcegroup-> -n <-name-> -o json`
- resource group - your resource group
- name - your custom name for new identity

**IMPORTANT:**
Remember two parameters from output: **clientId** and **id**.

4. Assign **Managed Identity Operator** role for your identity:

`az role assignment create --role "Managed Identity Operator" --assignee <-sp_aks_id->  --scope <-full_id->`
- sp_aks_id - AKS Service Principal. (Execute `az aks list`, find your aks cluster and grab `servicePrincipalProfile.clientId` field)
- full_id - id from step 3

5. Assign **Reader** role for your identity to KeyVault:
   
`az role assignment create --role Reader --assignee <-client_id-> --scope /subscriptions/<-subscription_id->/resourcegroups/<-rg_name->/providers/Microsoft.KeyVault/vaults/<-kv_name->`
   - client_id - clientId from step 3
   - subscription_id - name of the subscription (can be read from portal, on "overview" page of your resource group)
   - rg_name - your resource group name
   - kv_name - your key vault name

6. Add access policy to your identity so it can read secret from key vault:
   -  In portal, go to your resource group and select your Key Vault servcice.
   -  From the left pane select **Access policies**
   -  Click **Add access policy** button
   -  **Secret permissions** - select **Get**
   -  **Select principal** - select your identity created in step 3
   -  Click **Add**
   -  Click **Save**

## Task 3: Set up resources on K8s cluster
1. Edit **01_aadpi.yaml** and replace values:
   - mi_name - managed identity name from Task 2, step 3
   - clientId - from Task 2, step 3
   - subscription_id - name of the subscription (can be read from portal, on "overview" page of your resource group)
   - rg_name - your resource group name

2. Edit **02_binding.yaml** and replace values:
   - mi_name - managed identity name from Task 2, step 3

3. Edit **03_deployment.yaml** and replace values:
   - kv_name - name of your key vault
   - secret_name - name of key vault secret (Created in Task 2)
   - rg_name - name of your resource group
   - subscription_id - name of the subscription (can be read from portal, on "overview" page of your resource group)
   - tenant_id - execute `az account show` and grab tenant_id

## END LAB

<br><br>

<center><p>&copy; 2019 Chmurowisko Sp. z o.o.<p></center>
