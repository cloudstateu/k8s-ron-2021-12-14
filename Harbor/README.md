# Harbor private registry

## LAB Overview

In this lab you will deploy Harbor private registry on your K8s cluster. Harbor is a secure container images storage https://goharbor.io/

## Prerequisite: Install Helm

1. Follow the instructions: https://helm.sh/docs/intro/quickstart/

## Task 1: Deploy harbor

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


## Task 2: Push image to harbor

1. There are two ways of resolving this issue:
- Add harbor host to docker insecure registries (https://docs.docker.com/registry/insecure/)
- Download certificate from harbor (it is stored as a K8s secret) and add this cert to Docker trusted certificates (https://docs.docker.com/engine/security/certificates/)

2. As second option is more secure, we will go this way. Download certificate from harbor (from K8s secrets or web UI)

3. Add this cert to trusted certificates accoring to instructions here: https://docs.docker.com/engine/security/certificates/

4. Add new harbor user

5. type `docker login HARBOR_HOST` and log in using newly created user.

6. Go to harbor web ui and create new project

7. Assign developer role to the project for the user

8. Copy push commands and push azure-voting image into repository.

## Task 3: deploy application

1. Add certificate to every node in the cluster using `certds.yaml`
2. Change image name in `azure-vote-deployment.yaml`
3. Try to deploy
4. Create k8s secret for Harbor using robot account:
   `kubectl create secret docker-registry regcredpwc --docker-server=k8slab02vncr.azurecr.io --docker-username=k8slab02vncr --docker-password=KF=x8g8Yse6gqUBONpim21juyFzVMvpE`
5. Check that application is working on public IP address


## Task 4: Configure Jenkins CD process
1. In Jenkins, create credentials for user and password to Harbor (robot account)
2. Create service account in k8s, that will be used by Jenkins:
- `kubectl create serviceaccount jenkins-robot`
- `kubectl create clusterrolebinding jenkins-robot-binding --clusterrole=cluster-admin --serviceaccount=default:jenkins-robot`
- `kubectl get serviceaccount jenkins-robot -o go-template --template='{{range .secrets}}{{.name}}{{"\n"}}{{end}}'`
- `kubectl get secrets jenkins-robot-token-d6d8z -o go-template --template '{{index .data "token"}}' | base64 -d`
3. Create new credential in Jenkins of type **Secret text** and paste the resulting token there.
4. Configure new Jenkins job with two steps:
```
# build docker image and push
WEB_IMAGE_NAME="piotrharbor.westeurope.cloudapp.azure.com/test/azurevote:${BUILD_NUMBER}"
cd ./azure-vote
docker build -t $WEB_IMAGE_NAME .
docker login piotrharbor.westeurope.cloudapp.azure.com -u ${HUSER} -p ${HPASSWORD}
docker push $WEB_IMAGE_NAME
```
```
# update k8s deployment
WEB_IMAGE_NAME="piotrharbor.westeurope.cloudapp.azure.com/test/azurevote:${BUILD_NUMBER}"
kubectl set image deployment/azure-vote-front azure-vote-front=$WEB_IMAGE_NAME
```

## END LAB

<br><br>

<center><p>&copy; 2021 Chmurowisko Sp. z o.o.<p></center>
