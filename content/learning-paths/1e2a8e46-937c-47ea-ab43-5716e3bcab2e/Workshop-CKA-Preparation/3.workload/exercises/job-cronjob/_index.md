---
title: Job & Cronjobs
---

## Exercise

1. Create the specification of a Job running 3 pods in sequence. Each Pod contains a single container based on the *alpine:3.15* image, prints the current date in its standard output and then waits for 3 seconds

2. Run the Job

3. Get the logs of the 3 pods

4. Create a CronJob that prints the current date in the standard output every minute

5. Delete the Job and CronJob

## Documentation

- [https://kubernetes.io/docs/concepts/workloads/controllers/job/](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a Job running 3 pods in sequence. Each Pod contains a single container based on the *alpine:3.15* image, prints the current date in its standard output and then waits for 3 seconds

```
k create job my-job --image=alpine:3.15 --dry-run=client -o yaml -- /bin/sh -c "date && sleep 3" > job.yaml
```

Then modify the specification to specify 3 pods must run in sequence

```
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec: 
  parallelism: 1
  completions: 3
  template:
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - date && sleep 3
        image: alpine:3.15
        name: my-job
      restartPolicy: Never
```

2. Run the Job

```
k apply -f job.yaml
```

3. Get the logs of the 3 pods

The name of the job is auto set as the value of the pods' *job-name* label:

```
k logs -l job-name=my-job
```

4. Create a CronJob that prints the current date in the standard output every minute

```
k create cronjob my-cronjob --image=alpine:3.15 --schedule="* * * * *" -- date
```

5. Delete the Job and CronJob

```
k delete job my-job
k delete cj my-cronjob
```

</details>

