# Job

``` yaml title="job.yaml"
apiVersion: Batch/v1
kind: Job
metadata:
  name: pi
  namespace: pi
  labels:
    app: pi
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl:5.34.0
          command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
  # ttlSecondsAfterFinished: 10 # remove job after 10 seconds
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/controllers/job/)