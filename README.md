
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
To delete everything, 
```
oc -n knativetutorial delete -f event-source.yaml
oc -n knativetutorial delete -f event-sink.yaml
```
### Deploying the Apache Kafka cluster

To deploy Apache kafka inside our Knative cluster we use this command

```
oc create namespace kafka &&\
curl -L \
https://github.com/strimzi/strimzi-kafka-operator\
/releases/download/v0.16.2/strimzi-cluster-operator-v0.16.2.yaml \
  | sed 's/namespace: .*/namespace: kafka/' \
  | oc apply -n kafka -f -
```

With this, the strimzi cluster operator pod should be running in Kafka namespace. To check this use,

```
oc get pods -n kafka
```
Now that the strimzi operator is running, we spin up our Kafka cluster with one node with the entity operator and zookeeper. 
```
oc apply -f kafka-cluster.yaml
```
You can check the deployment using 
```
oc get pods -n kafka
```
You can see 3 new pods, Kafka broker, Zookeepr and Entity operator

Create a topic in Kafka whic we will use in the next tutorial
```
oc apply -f kafka-topic.yaml
```
 
### Deploying Knative events with Kafka source

We need to set all the CRD's, Service account, Clusterrole, rolebindings etc for the further step. 
This step deploys everything,
```
oc apply -f kafka-source.yaml 
```
Make sure you have **kafka-controller-manager** pod running in knative-sources namespace
```
oc get pods -n knative-sources
```
This will deploy the dispatcher, controller and webhook which will be needed for making the internal connections along with a kafka cluster with 1 zookeper agent and a broker. 

```
curl -L "https://github.com/knative/eventing-contrib/\
releases/download/v0.12.2/kafka-channel.yaml" \
 | sed 's/REPLACE_WITH_CLUSTER_URL/my-cluster-kafka-bootstrap.kafka:9092/' \
 | oc apply --filename -
 ```
 
Deploy the Knative Service (sink) 
```
oc apply -f eventing-hello-sink.yaml
```
Initially this deployment will scale up to 1 pod but as it is not getting any events, Knative will scale it down to zero. Now only when we send the events, this service will spin up. This deployment deosnt actually generate any service but in a real world case this could be anything from spinning up a container to spinning up a URL etc.

We create a Kafka source which links the topic which we created earlier with the sink (eventing hello).
```
oc -n knativetutorial apply -f kafka-event-source.yaml
```

Now we have a Kafka source which connects our topic to our sink. So whenever we recieve any new messages to our Kafka broker, the source will direct it to Knative Service (sink). 

To test this, lets spin up a producer which sends msgs to our Kafka source. For this navigate to the bin                    directory and run

```
bash kafka-producer.sh
```
In the terminal, write json formatted messages :
```
{"hello":"world"}

{"Kafka":"Knative}
```

We can check the logs of our eventing pod to check if the sink has those messages.

First get the pod name
```
oc get pods -n knativetutorial
```
Use the pod having name eventinghello-XXX to check logs.
```
watch oc logs -n knativetutorial -f <pod-name> -c user-container
```
To clean up
```
oc delete -n knativetutorial  -f kafka-event-source.yaml
oc delete -n knativetutorial  -f eventing-hello-sink.yaml
oc delete -n kafka -f kafka-topic.yaml
```
