# k8s-job-redis

# Purpose
Quick demo of kubernetes jobs and redis.

This is really based on [documented examples]( https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)

# Usage
The majority of this is copied directly from the link above, this is just intended to be shorthand.

## Requirements
- Docker for mac with kubernetes enabled.
- kubectl configured to use docker for mac with empty namespace.


## Start Redis
Stand up Redis and Redis Service

```
kubectl create -f ./redis-pod.yaml
kubectl create -f ./redis-service.yaml
```


## Feed Redis

Run redis image (has redis client) and start a bash shell in the image. You will need to press <Enter> at the client prompt.
`kubectl run -i --tty temp --image redis --command "/bin/sh"`


Start the cli

`redis-cli -h redis`

Save some data under the "job2" key (I had to copy and paste 1 by 1)

```
rpush job2 "apple" 
rpush job2 "banana"
rpush job2 "cherry"
rpush job2 "date"
rpush job2 "fig"
rpush job2 "grape"
rpush job2 "lemon"
rpush job2 "melon"
rpush job2 "orange"
```

# List all elements stored under "job2" key
`lrange job2 0 -1`

Once completed, opend a new shell or close this one (Redis is fed)

## Run Job
Run the actual kubernetes job

`kubectl create -f ./job.yaml`


## Check output
Check the output of the kubernetes jobs.

To do this, first get the individual job names:

```
$ kubectl describe jobs/job-wq-2
Name:             job-wq-2
Namespace:        default
Selector:         controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
Labels:           controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
                  job-name=job-wq-2
Annotations:      <none>
Parallelism:      2
Completions:      <unset>
Start Time:       Mon, 11 Jan 2016 17:07:59 -0800
Pods Statuses:    1 Running / 0 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=b1c7e4e3-92e1-11e7-b85e-fa163ee3c11f
                job-name=job-wq-2
  Containers:
   c:
    Image:              gcr.io/exampleproject/job-wq-2
    Port:
    Environment:        <none>
    Mounts:             <none>
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  33s          33s         1        {job-controller }                Normal      SuccessfulCreate  Created pod: job-wq-2-lglf8
```

Then check the individual logs for each job pod (prepend `pods/` to the pod name from the message field above).

```
$ kubectl logs pods/job-wq-2-7r7b2
Worker with sessionID: bbd72d0a-9e5c-4dd6-abf6-416cc267991f
Initial queue state: empty=False
Working on banana
Working on date
Working on lemon
```



