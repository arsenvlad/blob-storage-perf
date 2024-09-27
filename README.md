# Azure Storage - Premium Block Blob Performance

Azure CLI login

```bash
az login --use-device-code --tenant 62e9dca3-7396-48ed-a055-61c12fc24020
az account set --subscription 46ce15d0-e6f3-424b-8ccb-2b41ec07d780
```

Create premium block blob account

```bash
az group create --name rg-storage1 --location eastus2
az storage account create --resource-group rg-storage1 --name avstoragepremium1 --location eastus2 --sku Premium_ZRS --kind BlockBlobStorage --allow-shared-key-access false --allow-blob-public-access false
az storage container create --auth-mode login --name azcopy-benchmark --account-name avstoragepremium1
```

Create standard storage account

```bash
az storage account create --resource-group rg-storage1 --name avstoragestandard1 --location eastus2 --sku Standard_ZRS --kind StorageV2 --allow-shared-key-access false --allow-blob-public-access false
az storage container create --auth-mode login --name azcopy-benchmark --account-name avstoragestandard1
```

Connect to an existing AKS cluster in the same region as the storage accounts

```bash
az aks get-credentials --resource-group rg-aksazurecilium1 --name aksazurecilium1 --subscription 1412f248-f41c-4c92-be6c-28f2700d1037

kubectl get nodes
kubectl get pods -o wide

kubectl apply -f azcopy.yaml
# kubectl delete -f azcopy.yaml
```

Exec into the pod and install AzCopy

```bash
kubectl exec -it azcopy-pod -- /bin/bash

apt-get update
apt-get install curl -y

curl -sSL -O https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb
dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
apt-get update
apt-get install azcopy
```

Run AzCopy benchmark against premium and standard storage accounts

```bash
azcopy login --tenant-id 62e9dca3-7396-48ed-a055-61c12fc24020
azcopy login status
azcopy bench --mode=Upload "https://avstoragepremium1.blob.core.windows.net/azcopy-benchmark" --file-count 500 --delete-test-data=false
azcopy bench --mode=Download "https://avstoragepremium1.blob.core.windows.net/azcopy-benchmark"

azcopy bench --mode=Upload "https://avstoragestandard1.blob.core.windows.net/azcopy-benchmark" --file-count 500 --delete-test-data=false
azcopy bench --mode=Download "https://avstoragestandard1.blob.core.windows.net/azcopy-benchmark"
```

## Screenshots of results

### Premium_ZRS

Upload
![a](/images/premiumzrs-upload.png)
![b](/images/premiumzrs-upload-latency.png)

Download
![c](/images/premiumzrs-download.png)

### Standard_ZRS

Upload
![d](/images/standardzrs-upload.png)
![e](/images/standardzrs-upload-latency.png)

Download
![f](/images/standardzrs-download.png)
