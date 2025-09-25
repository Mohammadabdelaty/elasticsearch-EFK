# EFK Stack

In this repo we will make a small project to be deployed in your home lab for testing.

## Elasticsearch

Use helm to deploy elasticsearch cluster with only one master node
```bash
helm repo add elastic https://helm.elastic.co

helm install elasticsearch elastic/elasticsearch \
  --namespace elasticsearch \
  --set replicas=1 \
  --set resources.requests.memory=1Gi \
  --set resources.requests.cpu=500m \
  --set volumeClaimTemplate.resources.requests.storage=5Gi
```
-------------------
## Elasticsearch Operator
This part in case you need to install an operator:

```bash
# apply pv
kubectl -n elasticsearch apply -f backup-pv.yaml

# patc pv with hostpath
kubectl patch pv es-local-pv-master -p '{"spec":{"storageClassName":"hostpath"}}'
kubectl patch pv es-local-pv-data -p '{"spec":{"storageClassName":"hostpath"}}'
kubectl patch pv es-local-pv-client -p '{"spec":{"storageClassName":"hostpath"}}'

# apply cluster
kubectl -n elasticsearch apply -f cluster.yaml
```
-------------------------

Get admin username and password to use in the following steps
```bash
kubectl get secrets --namespace=elasticsearch elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
kubectl get secrets --namespace=elasticsearch elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```

## Kibana (Dashboard)

```bash
helm install kibana elastic/kibana \
  --namespace elasticsearch \
  --set service.type=LoadBalancer
```

Then open http://localhost:5601 and try to acces using the cresd for the last step.

## FluenBit

```bash
helm install fluent-bit fluent/fluent-bit -f values.yaml -n elasticsearch
```

Now in the UI kibana go to: Stack Management > Data View > Add Data. Name it, Then make the index is logstash -just copy logstash line from the right column.

![alt text](screens/image.png)

While adding the data you should see index pattern: `logstash-2025.09.24` 

![alt text](screens/image-1.png)

If not, so there is an issue with flunetbit sending logs to elastic

![alt text](screens/image-2.png)

Then start watching you logs

![alt text](screens/image-3.png)

## EBS: Snapshots

