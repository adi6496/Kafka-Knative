
# Apache Kafka with Knative eventing

## Prerequisites
1. Access to openshift cluster
2. Knative installed on openshift
3. Knative serving installed
4. Knative eventing installed

Create a new project in openshift

```
oc new-project knativetutorial
```
## Knative service example

### 1. Deploying a Knative service

Navigate to Serving folder Knative-Serving

Deploy a Knative service which has an image which outputs **Hi greeter ⇒ '6fee83923a9f' : 1** on the command line

To deploy, 
```
oc apply -f service.yaml -n knativetutorial
```

To check for successful eployment,
```
oc get pods -n knativetutorial
```
After successful deployment, we can see the pod similar to *greeter-xxxx-deployment-xxx* 

To invoke the servie, we first need to get the URL of the service. 

```
export SERVICEURL=`oc get rt greeter -o yaml | yq read - 'status.url'`
```

Invoke it using,
```
http $SERVICEURL
```
The http command should return a response which has a line similar to **Hi greeter ⇒ '6fee83923a9f' : 1** in the end
