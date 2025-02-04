# 本方案目的
   - 旨在解决 `Azure Portal` 手工部署 `qdrant container apps` 运行一段时间数据消失的问题
     - 不确定原因，高度怀疑 `Azure conatiner apps` By Design 运行不带卷 Volume 导致的
       - 未发现自动重启迹象
         - `Azure conatiner apps` 也没有 `restart` 功能，只有 `delete` 功能
       - `qdrant` 确实需要静态稳定的存储
     - Qdrant Vector DB with Volume on Azure
       - Azure 上带有卷的 Qdrant Vector DB
         
         https://github.com/AwesomeYuer/qdrant-azure-arm-deployment/tree/main/Azure-Container-Apps#qdrant-vector-db-with-volume-on-azure
         
         ![image](https://github.com/AwesomeYuer/qdrant-azure-arm-deployment/assets/1026479/290b3c85-57a4-4ca8-91ba-d0014c9a2d19)

       - 利用如下 `arm` 魔板部署 `conatiner apps`，可能是因为带卷 Volume, 也指定了专用的 `File Storage Account` 所以不再丢失数据

# 安全起见，配置使用 `api-key` 访问
  - 修改魔板
    - 参阅

      https://qdrant.tech/documentation/guides/configuration/

      https://github.com/AwesomeYuer/qdrant-azure-arm-deployment/tree/main/Azure-Container-Apps

    - 相当于容器运行方式指定环境变量参数 
  
        ```sh
        docker run -e api_key=xxxxxxxxxxxxxxxxx
        ```
        
    - 魔板指定 `QDRANT__SERVICE__API_KEY` 环境变量, 限制为使用 `api-key` 访问
   
      ```json
      {
        "template": {
          "containers": [
            {
              "name": "qdrantapicontainerapp",
              "image": "qdrant/qdrant",
              "resources": {
                "cpu": 2,
                "memory": "4Gi"
              },
              "volumeMounts": [
                {
                  "volumeName": "qdrantstoragevol",
                  "mountPath": "/qdrant/storage"
                }
              ],
              "env": [
                {
                  "name": "QDRANT__SERVICE__API_KEY",
                  "value": "!@#123QWEqwe"
                }
              ]
            }
          ]
        }
      }
      ```

# 为 Azure `qdrant container apps` 配置 `网络安全组(NSG)` 入栈规则
 - 刚运行时，内网其他应用跨 `VNET` 访问正常，过一段时间就访问不了了

   ![image](https://github.com/AwesomeYuer/qdrant-azure-arm-deployment/assets/1026479/fafdc369-8dc8-4ff7-baed-73c4509693b7)


# 一键点击下面按钮
  
  - 修改魔板
  - 部署 `qdrant container apps`:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAwesomeYuer%2Fqdrant-azure-arm-deployment%2Fmain%2FAzure-Container-Apps%2FARM-templates%2Fqdrant-aca-deploy.json)


-------------------------------------------------------

# Qdrant Vector Database on Azure Cloud

<img src="./img/qdrant-plus-azure.png" width="480" height="300" />

This project combines the power of the Qdrant Vector Database with the Microsoft Azure Cloud
allowing you to bring Vector Search and Embeddings storage to your AI products.

## Getting started

You have several options for how to get Qdrant running on Azure:

- [Azure Kubernetes Service](Azure-Kubernetes-Svc/README.md)
- [Qdrant Container in Docker](Local-Docker-Deployment/README.md)

## Prerequisites

To get started, users will need access to an Azure subscription.

To deploy using the Deploy to Azure button which leverages an ARM template, you need write access on the resources you're deploying and access to all operations on the Microsoft.Resources/deployments resource type.

## Installation

### Azure Kubernetes Service

To deploy Qdrant to a cluster running in Azure Kubernetes Services, go to the `Azure-Kubernetes-Svc` folder and follow instructions in the `README.md` to deploy to a Kubernetes cluster with Load Balancer on Azure Kubernetes Services (AKS).

You can quickly create an **Azure Kubernetes Service** cluster by clicking the Deploy to Azure button below. After creating your AKS cluster, go to the `Azure-Kubernetes-Svc` folder to deploy **Qdrant** into the AKS cluster using **Helm**.

#### AKS Prerequisites 
**PLEASE NOTE! ensure that you have a resource group and ssh key in that resource created before selecting the Deploy to Azure Button below. More details can be found in README.md in [Azure Kubernetes Service README.md](Azure-Kubernetes-Svc/README.md)**

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fqdrant-azure%2Fmain%2FAzure-Kubernetes-Svc%2Faks-arm-deploy.json)

### Docker (Local)

To develop against and run Qdrant locally. 

**VS Code Dev Container**
This project contains a dev container configuration which can be used for local development. To learn more about using a dev container, please review [Developing inside a Container](https://code.visualstudio.com/docs/devcontainers/containers).

**Docker Locally**
To run the **Qdrant** vector database running in **Docker** locally, please follow the instructions from Qdrant's website:
[Install Qdrant with Docker](https://qdrant.tech/documentation/install/#with-docker)

To run Qdrant with Docker locally, you can use the following command using  default values stored in the file `.config/config.yaml` located in the `Local-Docker-Deployment` folder.

```bash
docker run -p 6333:6333 \
    -v $(pwd)/path/to/data:/qdrant/storage \
    -v $(pwd)/path/to/custom_config.yaml:/qdrant/config/production.yaml \
    qdrant/qdrant
```

You can overwrite values by creating and adding new records to a file `./config/production.yaml`. An example of the production.yaml file located in the `Local-Docker-Deployment` directory. Please review the [Qdrant documentation](https://qdrant.tech/documentation/install/#configuration) to learn more information on configuration options for **Qdrant**.

## Resources for Learning More

- [Semantic Kernel Blog: The Power of Persistent Memory with Semantic Kernel and Qdrant Vector Database](https://devblogs.microsoft.com/semantic-kernel/the-power-of-persistent-memory-with-semantic-kernel-and-qdrant-vector-database/)
- [Qdrant Vector Search [Vector Database]](https://qdrant.tech/)
- [Qdrant Integration with OpenAI](https://qdrant.tech/documentation/integrations/#openai)
- [Azure Container Instances (ACI)](https://learn.microsoft.com/azure/container-instances/)
- [Azure Kubernetes Service (AKS)](https://learn.microsoft.com/azure/aks/)
- [Docker Desktop](https://docs.docker.com/desktop/)
