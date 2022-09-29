# CronJob

``` yaml title="cronjob.yaml"
apiVersion: Batch/v1
kind: CronJob
metadata:
  name: pi
  namespace: pi
  labels:
    app: pi
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
              - /bin/sh
              - -c
              - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/controllers/cron-jobs/)

[Crontab Guru](https://crontab.guru/)