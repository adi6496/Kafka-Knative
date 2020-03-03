
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

To delete the service, 
```
oc -n knativetutorial delete services.serving.knative.dev greeter
```
### 2. Knative Eventing with a CronJob Source 
Event sources emit out events which can then be recieved by the sink. 

Deploy the Cronjob source, which emits events (data) according to the schedule mentioned.

```
oc apply -f event-source.yaml -n knativetutorial
```

Now we create a sink, where we recieve these events.
```
oc apply -f event-sink.yaml -n knativetutorial
```

You can watch the logs on a different terminal. 

To watch logs, get the name of eventing pod using this,
```
oc get pods -n knativetutorial
```
and then use this to watch logs of eventing podpod
```
oc logs -n knativetutorial -f <pod-name> -c user-container
```

