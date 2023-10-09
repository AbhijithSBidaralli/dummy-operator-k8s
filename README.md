# dummy-operator-k8s
Operator SDK is a powerful toolkit that simplifies the development of Kubernetes Operators, enabling the creation of custom controllers to manage complex applications and custom resources in a Kubernetes environment. This project explores the fundamental concepts behind Operator SDK and its usage in building custom controllers. It highlights the key steps involved, including defining custom resources, generating code scaffolds, implementing reconciliation logic, and testing controllers. By leveraging Operator SDK, developers can streamline the process of building and managing their custom controllers, enhancing the management of their applications and resources within Kubernetes clusters.

In this project we will use the Operator SDK to:
1. Create a new custom API type called ”Dummy”.
2. Create a new custom controller for resources of kind ”Dummy”.

”Dummy” resources should have a spec field that has only
one string subfield called ”message”.

We will then extend the Dummy custom API type by giving to it a status field which
contains only a string subfield called specEcho and then copy the value of ***spec.message*** into ***status.specEcho***.

After a Dummy has been processed by the custom controller it
should look like:

``` 
apiVersion: interview.com/v1alpha1
kind: Dummy
metadata:
    name: dummy1
    namespace: default
spec:
    message: "I'm just a dummy"
status:
    specEcho: "I'm just a dummy
``` 

## Requirements and Installation
Following dependencies recommended:

- Docker
- Kubernetes and Kubectl
- Go Lang (version go1.21.1)
- Operator-SDK (v1.31.0)

## Steps to create your own operator

### Step 1: Create a new operator-sdk project
```bash
operator-sdk init --domain interview.com --repo github.com/AbhijithSBidaralli/dummy-operator-k8s
``` 
### Step 2: Create a new API and controller
```bash
operator-sdk create api --group api --version v1alpha1 --kind Dummy
``` 
This will generate the boilerplate code. The generated code includes a controller named dummy_controller.go and also dummy_types.go file.

### Step 3: Make changes to boilerplate code
The first step is configuring our resource to have fields related to spec. To do this, you must change the [`api/v1alpha/dummy_types.go`](/api/v1alpha1/dummy_types.go) file and add the desired fields.
The next step is to create the logic for our controller in the [`controllers/dummy_controller.go`](/controllers/dummy_controller.go) file and update the ‌Reconcile function.
We will also update the [`config/samples/api_v1alpha1_dummy.yaml`](/config/samples/api_v1alpha1_dummy.yaml) file accordingly.
Once you make these changes run the below command:
```bash
make manifests
```
**Note**: Always run above command whenever you make any changes to code.

### Step 4: Install the Custom Resource Definition/Controller
```bash
kubectl apply -f config/crd/bases/api.interview.com_dummies.yaml
```
You should get following output
```cmd
customresourcedefinition.apiextensions.k8s.io/dummies.api.interview.com created
```
You can run below command to check if it has been installed succesfully:
```bash
kubectl get crd
```
### Step 5: Run the controller
```bash
make run
```
### Step 6: Apply the yaml with the Dummy definition
```bash
kubectl apply -f config/samples/api_v1alpha1_dummy.yaml
```
when you run
```bash
kubectl describe  dummy.api.interview.com 
```
you should see following result:
```console
Name:         dummy1
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  api.interview.com/v1alpha1
Kind:         Dummy
Metadata:
  Creation Timestamp:  2023-10-08T20:22:19Z
  Generation:          1
  Resource Version:    11511
  UID:                 b2580655-1f1d-4219-b51b-f7fbeb514632
Spec:
  Message:  I'm just a dummy!
Status:
  Spec Echo:  I'm just a dummy!
Events:       <none>

```
To delete the controller use
```bash
kubectl delete -f config/crd/bases/api.interview.com_dummies.yaml
```
### Step 7: Build the docker image and push to docker hub
```bash
make docker-build docker-push IMG=registry.hub.docker.com/abhijithbidaralli/dummy-operator-k8s:latest
```
### Step 8: Install generated container and check if it's working
```bash
make deploy IMG=registry.hub.docker.com/abhijithbidaralli/dummy-operator-k8s:latest
```
Run this command in separate terminal and you should get same output as before
```bash
kubectl apply -f config/samples/api_v1alpha1_dummy.yaml
```
Delete using
```bash
make undeploy
```
