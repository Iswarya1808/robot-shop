  # Microservices CI/CD & Mesh: Stan's Robot Shop on Azure Kubernetes Services

This project demonstrates a production-grade deployment of the Stan's Robot Shop microservice application. It leverages a full DevOps suite including SonarCloud, Trivy,GitHub Self-Hosted runners, and a robust Istio Service Mesh for observability.


## Technologies used:
- NodeJS ([Express](http://expressjs.com/))
- Java ([Spring Boot](https://spring.io/))
- Python ([Flask](http://flask.pocoo.org))
- Golang
- PHP (Apache)
- MongoDB
- Redis
- MySQL ([Maxmind](http://www.maxmind.com) data)
- RabbitMQ
- Nginx
- AngularJS (1.x)

## Build from Source
* Cloud: Azure (AKS, ACR)
* CICD: GitHub Actions with Self-Hosted Runner
* Security: SonarCloud (SAST) & Trivy (Container Scanning)
* Service Mesh: Istio (mTLS, Traffic Management)
* Observability: Kiali, Prometheus and Grafana

## Prerequisites
Before starting, ensure you have:
1. An active Azure Subscription
2. GitHub Self hosted runner (Org level or Repo level)
3. SonarCloud account and token
4. `kubectl`,`helm` and `az cli` installed on the runner.

## Deployment Steps
1. Infrastructure Setup
   
Login to the Azure and Create the base resources:

```shell
$ az group create --name RobotShopRG --location eastus # Created Resource Group
$ az acr create --resource-group RobotShopRG --name <ACRname> --sku Standard # Creating Container Registry

# Creating AKS Cluster
$ az aks create --resource-group RobotShopRG --name RobotShopekscluster --node-count 3 --generate-ssh-key --attach-acr <ACRname>

#Initial Helm deployment
$ helm install myapp ./k8s/helm
$ helm upgrade --install myapp ./k8s/helm  # if you already have a release.
```
2. CI/CD Pipeline Flow
   
The pipeline is defined in `.github/workflows/push.yml` and executes the following:

   1. **SonarCloud Scan**: Analyze code quality for the microservices.
   2. **Docker Build**: Builds images locally on the self-hosted runner.
   3. **Trivy Scan**: Scans images for vulnerabilities. Fails the build if `Critical` and `High` issues exist.
   4. **ACR Push**: Pushes verified images to Azure Container Registry.
   5. **Helm Deploy**: Deploys/Updates the application in AKS.
      
3. Service Mesh & Observability
   Once the cluster is up, install the Istio service mesh and monitoring stack:
   ```shell
    # Install Istio
    $ istioctl install --set profile=demo -y

    # Enable Istio Injection for the app namespace
    $ kubectl label namespace default istio-injection=enabled

    # Install Observability Stack (Prometheus , Grafana and Kiali)
    $ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    $ helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create namespace
    $ helm install kiali-server kiali/kiali-server -n istio-system
    ```
4. Exposing the Application
   Apply the **Istio Gateway** to allow external traffic to the Robot Shop web interface:
   ```shell
      $ kubectl apply -f K8s/Istio/gateway.yml
      ```
## Enabled Endpoints 
The **cart** and **payment** services both have Prometheus metric endpoints. These are accessible on **`/metrics`**. The cart service provides:
To test the metrics use:

```shell
$ curl http://<host>:8080/api/cart/metrics
$ curl http://<host>:8080/api/payment/metrics
```

## Observability Dashboards

Access the following tools to monitor the communications
| Tools                                              | Access Command      | Description                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Kiali                           | istioctl dashboard kiali          | Traffic topology & mTLS status |
| Grafana                     | kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring           | Visualizing performance metrics                                                        |
| Prometheus | kubectl port-forward svc/monitoring-prometheus 8080:8080 -n monitoring          | Raw metrics data  |

## Security Best Practices
* mTLS: Enabled by default via Istio for all service-to-service communication.
* Container Scanning: Every image is scanned by Trivy before being allowed into the ACR.
* SAST: SonarCloud gates ensure no technical debt or secrets are committed to the repositary.

## Running Locally

To download the tracing module for Nginx, it needs a valid Instana agent key. Set this in the environment before starting the build.

```shell
$ export INSTANA_AGENT_KEY="<your agent key>"
```

Now build all the images.

```shell
$ docker-compose build
```

If you modified the `.env` file and changed the image registry, you need to push the images to that registry

```shell
$ docker-compose push
```
Run Locally
You can run it locally for testing.

If you did not build from source, don't worry all the images are on Docker Hub. Just pull down those images first using:

```shell
$ docker-compose pull
```

Fire up Stan's Robot Shop with:

```shell
$ docker-compose up
```

If you want to fire up some load as well:

```shell
$ docker-compose -f docker-compose.yaml -f docker-compose-load.yaml up
```

If you are running it locally on a Linux host you can also run the Instana [agent](https://docs.instana.io/quick_start/agent_setup/container/docker/) locally, unfortunately the agent is currently not supported on Mac.

There is also only limited support on ARM architectures at the moment.

## Contact & Support
If you have questions about the configuration, feel free to open an issue or reach out via LinkedIn. 
www.linkedin.com/in/sharnitha-vijayakumar-b124701a7
