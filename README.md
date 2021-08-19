# nginx_basicauth_demo

Prerequisites:

* GKE cluster created
* Obtain GKE cluster credentials
* Create a kubernetes namespace for the nginx instances

Set the following variables. Note: Retain the single-quotes around the password entries.

```bash
export NOAUTH_REG_NAME=<REGISTRY_NAME>
export AUTH_REG_NAME=<REGISTRY_NAME>
export USER1_PASS='<PASSWORD>'
export USER2_PASS='<PASSWORD>'
export USER3_PASS='<PASSWORD>'
export K8S_NAMESPACE=<K8S_NAMESPACE_NAME>
```

Build the nginx-noauth container image

```bash
cd nginx-noauth
docker build -t nginx-noauth .
```

Tag the nginx-noauth container image and push to a container registry 

```bash
docker tag nginx-noauth:latest ${NOAUTH_REG_NAME}/nginx-noauth:v1 
docker push ${NOAUTH_REG_NAME}/nginx-noauth:v1
```

Build the nginx-auth container image

```bash
cd ../nginx-auth
docker build -t nginx-auth --build-arg USER1_PASS=${USER1_PASS} --build-arg USER2_PASS=${USER2_PASS} --build-arg USER3_PASS=${USER3_PASS} .
```

Tag the nginx-auth container image and push to a container registry

```bash
docker tag nginx-auth2:latest ${AUTH_REG_NAME}/nginx-auth:v1
docker push ${AUTH_REG_NAME}/nginx-auth2:v1
```

Update the Kubernetes manifiest files with appropriate image path

```bash
cd ..
sed -i '' 's/<REPLACE_WITH_REG_NAME>/'${NOAUTH_REG_NAME}'/g' "$(pwd)/k8s-manifests/nginx-noauth.yaml"
sed -i '' 's/<REPLACE_WITH_REG_NAME>/'${AUTH_REG_NAME}'/g' "$(pwd)/k8s-manifests/nginx-auth.yaml"
```

Deploy the kubernetes manifests

```bash
kubectl apply -f k8s-manifests/nginx-noauth.yaml -n ${K8S_NAMESPACE}
kubectl apply -f k8s-manifests/nginx-auth.yaml -n ${K8S_NAMESPACE}
```

Monitor the deployments and record the services' external IP address

```bash
kubectl get all -n ${K8S_NAMESPACE}
```

Open a browser windows and test access to each of the nginx services' external IP address; ex. **http://<EXTERNAL_IP_ADDRESS>**. The noauth service endpoint will send you directly to the main page while the auth service endpoint will prompt you for a user name and password before permitting access to the main page. For the auth service endpoint prompt, enter user1, user2, or user3 and the respective password set in the variable.  

For each service endpoint create uptime check in Cloud Monitoring. Enter the **<EXTERNAL_IP>** as the URL and for the auth service uptime check, enter the username/password combination in the **Authentication** section.

Test the auth service uptime check by changing the user password in the nginx container

```bash
kubectl get pods -n ${K8S_NAMESPACE}
kubectl exec -it pod/nginx-auth-<UUID> bash
cat /etc/apache2/.htpasswd
htpasswd -cb /etc/apache2/.htpasswd user1 'P@$$w0rdxyz'
```

Monitor the aut service uptime check for degradation and eventual failure. Then, restore the uptime check by repeating the previous command with the correct password.
