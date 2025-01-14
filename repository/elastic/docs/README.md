# Elastic

Elasticsearch is a distributed, RESTful search and analytics engine. It is based on Apache Lucene.

This Framework is deploying an Elasticsearch Cluster.

## Prerequisites

You need a `Kubernetes cluster` up and running and `Persistent Storage` available with a default `Storage Class` defined.

If you use `minikube` then launch it with the following resource options.

```
minikube start --vm-driver=hyperkit --cpus=3 --memory=9216 --disk-size=10g
```


## Deploy an Instance

Deploy the `Instance` using the following command:
```
kubectl kudo install elastic --instance myes
```

Once the deployment has finished use the following command.
```
kubectl get pods
```

You should see that 3 master, 2 data, and 1 coordinator node have been deployed.

```
NAME                 READY   STATUS    RESTARTS   AGE
myes-coordinator-0   1/1     Running   0          23m
myes-data-0          1/1     Running   0          24m
myes-data-1          1/1     Running   0          24m
myes-master-0        1/1     Running   0          25m
myes-master-1        1/1     Running   0          24m
myes-master-2        1/1     Running   0          24m
```

## Use the Instance

Exec into one of the POD's.
```
kubectl exec -ti myes-master-0 bash
```

Use the following curl command to check the health of the cluster.
```
curl myes-coordinator-0.myes-coordinator-hs:9200/_cluster/health?pretty
```

You should see the following output.
```
{
  "cluster_name" : "myes-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 6,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

Lets add some data.
```
curl -X POST "myes-coordinator-0.myes-coordinator-hs:9200/twitter/_doc/" -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

Lets search for the entry.
```
curl -X GET "myes-coordinator-0.myes-coordinator-hs:9200/twitter/_search?q=user:kimchy&pretty"
```

You should see the following output.
```
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "twitter",
        "_type" : "_doc",
        "_id" : "n18aemoBCj0qv5VrMWv2",
        "_score" : 0.2876821,
        "_source" : {
          "user" : "kimchy",
          "post_date" : "2009-11-15T14:12:12",
          "message" : "trying out Elasticsearch"
        }
      }
    ]
  }
}
```

You can learn more on how to use elasticsearch from the [elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html).

## Update the Instance

Lets increase the `DATA_NODE_COUNT` to `3` using the following command.

```
kubectl patch instance myes -p '{"spec":{"parameters":{"DATA_NODE_COUNT":"3"}}}' --type=merge
```

Lets check on the pods.
```
kubectl get pods
```

You should see that a `3rd data` node has been deployed.
```
NAME                 READY   STATUS    RESTARTS   AGE
myes-coordinator-0   1/1     Running   0          3m37s
myes-data-0          1/1     Running   0          4m2s
myes-data-1          1/1     Running   0          3m49s
myes-data-2          1/1     Running   0          73s
myes-master-0        1/1     Running   0          4m30s
myes-master-1        1/1     Running   0          4m21s
myes-master-2        1/1     Running   0          4m13s
```


## Upgrade the Instance

... more to come ...
