# Harbor private registry

## LAB Overview

In this lab you will deploy Harbor private registry on your K8s cluster. Harbor is a secure container images storage https://goharbor.io/

## Prerequisite: Install Helm

1. Follow the instructions: https://helm.sh/docs/intro/quickstart/

## Task 1: Create new namespace

1. Deploy Harbor

We want the harbor to be publicly accessible via domain name. First you need to replace {UNIQUE_NAME} with the subdomain name for westeurope.cloudapp.azure.com.

westeurope.cloudapp.azure.com is a domain name for Azure Load Balancer that is assigned for K8s. We will need to assign this subdomain name for Harbor's public ip. But before we deploy the Harbor we don't have this IP assigned yet. That is why we need to guess a subdomain name and hope it is unique across westeurope.cloudapp.azure.com.

    ```bash
    helm install my-harbor harbor/harbor \ 
    --version 1.7.3 --set expose.type=loadBalancer \
    --set expose.tls.auto.commonName={UNIQUE_NAME}.westeurope.cloudapp.azure.com \
    --set externalURL=https://{UNIQUE_NAME}.westeurope.cloudapp.azure.com
    ```
Wait for the deployment to finish

2. Execute 
```
kubectl get svc
```
and find ip address of Harbor service.

3. Go to Azure portal and find this IP address service. (It will be on K8s's managed resource group).

4. Set domain name for the ip address using {UNIQUE_NAME} that you used in step 1.

5. Go to {UNIQUE_NAME}.westeurope.cloudapp.azure.com

6. Log in using credentials:
```
user: admin
pass: Harbor12345
```

7. The harbor is all set up however you won't be able yet to push an image there. Why? Because the harbor's ssl certificate is issued by unauthorized CA (created during harbor deployment).

8. There are two ways of resolving this issue:
- Add harbor host to docker insecure registries (https://docs.docker.com/registry/insecure/)
- Download certificate from harbor (it is stored as a K8s secret) and add this cert to Docker trusted certificates (https://docs.docker.com/engine/security/certificates/)


## END LAB

<br><br>

<center><p>&copy; 2021 Chmurowisko Sp. z o.o.<p></center>
