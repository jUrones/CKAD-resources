# Solved exercises CKAD exam 2024
Question 1 | Namespaces
-----------------------

Solve this question on instance: `ssh ckad5601`

The DevOps team would like to get the list of all _Namespaces_ in the cluster. Get the list and save it to `/opt/course/1/namespaces` on `ckad5601`.

##### Answer:

k get ns > /opt/course/1/namespaces

The content should then look like:

xxxxxxxxxx

\# /opt/course/1/namespaces

NAME              STATUS   AGE

default           Active   136m

earth             Active   105m

jupiter           Active   105m

kube-node-lease   Active   136m

kube-public       Active   136m

kube-system       Active   136m

mars              Active   105m

shell-intern      Active   105m

Question 2 | Pods
-----------------

Solve this question on instance: `ssh ckad5601`

Create a single _Pod_ of image `httpd:2.4.41-alpine` in _Namespace_ `default`. The _Pod_ should be named `pod1` and the container should be named `pod1-container`.

Your manager would like to run a command manually on occasion to output the status of that exact _Pod_. Please write a command that does this into `/opt/course/2/pod1-status-command.sh` on `ckad5601`. The command should use `kubectl`.

##### Answer:

xxxxxxxxxx

k run \# help

k run pod1 \--image\=httpd:2.4.41-alpine \--dry-run\=client \-oyaml > 2.yaml

vim 2.yaml

Change the container name in `2.yaml` to `pod1-container`:

xxxxxxxxxx

\# 2.yaml

apiVersion: v1

kind: Pod

metadata:

  creationTimestamp: null

  labels:

    run: pod1

  name: pod1

spec:

  containers:

  - image: httpd:2.4.41-alpine

    name: pod1-container \# change

    resources: {}

  dnsPolicy: ClusterFirst

  restartPolicy: Always

status: {}

Then run:

xxxxxxxxxx

➜ k create -f 2.yaml

pod/pod1 created

➜ k get pod

NAME   READY   STATUS              RESTARTS   AGE

pod1   0/1     ContainerCreating   0          6s

➜ k get pod

NAME   READY   STATUS    RESTARTS   AGE

pod1   1/1     Running   0          30s

Next create the requested command:

xxxxxxxxxx

vim /opt/course/2/pod1-status-command.sh

The content of the command file could look like:

xxxxxxxxxx

\# /opt/course/2/pod1-status-command.sh

kubectl -n default describe pod pod1 | grep -i status:

Another solution would be using jsonpath:

xxxxxxxxxx

\# /opt/course/2/pod1-status-command.sh

kubectl -n default get pod pod1 -o jsonpath="{.status.phase}"

To test the command:

xxxxxxxxxx

➜ sh /opt/course/2/pod1-status-command.sh

Running

Question 3 | Job
----------------

Solve this question on instance: `ssh ckad7326`

Team Neptune needs a _Job_ template located at `/opt/course/3/job.yaml`. This _Job_ should run image `busybox:1.31.0` and execute `sleep 2 && echo done`. It should be in namespace `neptune`, run a total of 3 times and should execute 2 runs in parallel.

Start the _Job_ and check its history. Each pod created by the _Job_ should have the label `id: awesome-job`. The job should be named `neb-new-job` and the container `neb-new-job-container`.

##### Answer:

xxxxxxxxxx

k \-n neptun create job \-h

k \-n neptune create job neb-new-job \--image\=busybox:1.31.0 \--dry-run\=client \-oyaml > /opt/course/3/job.yaml \-- sh \-c "sleep 2 && echo done"

vim /opt/course/3/job.yaml

Make the required changes in the yaml:

xxxxxxxxxx

\# /opt/course/3/job.yaml

apiVersion: batch/v1

kind: Job

metadata:

  creationTimestamp: null

  name: neb-new-job

  namespace: neptune  \# add

spec:

  completions: 3  \# add

  parallelism: 2  \# add

  template:

    metadata:

      creationTimestamp: null

      labels:             \# add

        id: awesome-job   \# add

    spec:

      containers:

      - command:

        - sh

        - \-c

        - sleep 2 && echo done

        image: busybox:1.31.0

        name: neb-new-job-container \# update

        resources: {}

      restartPolicy: Never

status: {}

Then to create it:

xxxxxxxxxx

k \-f /opt/course/3/job.yaml create \# namespace already set in yaml

Check _Job_ and _Pods_, you should see two running parallel at most but three in total:

xxxxxxxxxx

➜ k -n neptune get pod,job | grep neb-new-job

pod/neb-new-job-jhq2g              0/1     ContainerCreating   0          4s

pod/neb-new-job-vf6ts              0/1     ContainerCreating   0          4s

job.batch/neb-new-job   0/3           4s         5s

➜ k -n neptune get pod,job | grep neb-new-job

pod/neb-new-job-gm8sz              0/1     ContainerCreating   0          0s

pod/neb-new-job-jhq2g              0/1     Completed           0          10s

pod/neb-new-job-vf6ts              1/1     Running             0          10s

job.batch/neb-new-job   1/3           10s        11s

➜ k -n neptune get pod,job | grep neb-new-job

pod/neb-new-job-gm8sz              0/1     ContainerCreating   0          5s

pod/neb-new-job-jhq2g              0/1     Completed           0          15s

pod/neb-new-job-vf6ts              0/1     Completed           0          15s

job.batch/neb-new-job   2/3           15s        16s

➜ k -n neptune get pod,job | grep neb-new-job

pod/neb-new-job-gm8sz              0/1     Completed          0          12s

pod/neb-new-job-jhq2g              0/1     Completed          0          22s

pod/neb-new-job-vf6ts              0/1     Completed          0          22s

job.batch/neb-new-job   3/3           21s        23s

Check history:

xxxxxxxxxx

➜ k -n neptune describe job neb-new-job

...

Events:

  Type    Reason            Age    From            Message

  ----    ------            ----   ----            -------

  Normal  SuccessfulCreate  2m52s  job-controller  Created pod: neb-new-job-jhq2g

  Normal  SuccessfulCreate  2m52s  job-controller  Created pod: neb-new-job-vf6ts

  Normal  SuccessfulCreate  2m42s  job-controller  Created pod: neb-new-job-gm8sz

At the age column we can see that two `pods` run parallel and the third one after that. Just as it was required in the task.

Question 4 | Helm Management
----------------------------

Solve this question on instance: `ssh ckad7326`

Team Mercury asked you to perform some operations using Helm, all in _Namespace_ `mercury`:

1.  Delete release `internal-issue-report-apiv1`
    
2.  Upgrade release `internal-issue-report-apiv2` to any newer version of chart `bitnami/nginx` available
    
3.  Install a new release `internal-issue-report-apache` of chart `bitnami/apache`. The _Deployment_ should have two replicas, set these via Helm-values during install
    
4.  There seems to be a broken release, stuck in `pending-install` state. Find it and delete it
    

##### Answer:

_Helm Chart_: Kubernetes YAML template-files combined into a single package, _Values_ allow customisation

_Helm Release_: Installed instance of a _Chart_

_Helm Values_: Allow to customise the YAML template-files in a _Chart_ when creating a _Release_

**1.**

First we should delete the required release:

xxxxxxxxxx

➜ helm -n mercury ls

NAME                            NAMESPACE     STATUS          CHART           APP VERSION

internal-issue-report-apiv1     mercury       deployed        nginx-9.5.0     1.21.1     

internal-issue-report-apiv2     mercury       deployed        nginx-9.5.0     1.21.1     

internal-issue-report-app       mercury       deployed        nginx-9.5.0     1.21.1  

➜ helm -n mercury uninstall internal-issue-report-apiv1

release "internal-issue-report-apiv1" uninstalled

➜ helm -n mercury ls

NAME                            NAMESPACE     STATUS          CHART           APP VERSION

internal-issue-report-apiv2     mercury       deployed        nginx-9.5.0     1.21.1     

internal-issue-report-app       mercury       deployed        nginx-9.5.0     1.21.1  

**2**.

Next we need to upgrade a release, for this we could first list the charts of the repo:

xxxxxxxxxx

➜ helm repo list

NAME    URL                               

bitnami https://charts.bitnami.com/bitnami

➜ helm repo update

Hang tight while we grab the latest from your chart repositories...

...Successfully got an update from the "bitnami" chart repository

Update Complete. ⎈Happy Helming!⎈

➜ helm search repo nginx

NAME                  CHART VERSION   APP VERSION     DESCRIPTION          

bitnami/nginx         9.5.2           1.21.1          Chart for the nginx server             ...

Here we see that a newer chart version `9.5.2` is available. But the task only requires us to upgrade to any newer chart version available, so we can simply run:

xxxxxxxxxx

➜ helm -n mercury upgrade internal-issue-report-apiv2 bitnami/nginx

Release "internal-issue-report-apiv2" has been upgraded. Happy Helming!

NAME: internal-issue-report-apiv2

LAST DEPLOYED: Tue Aug 31 17:40:42 2021

NAMESPACE: mercury

STATUS: deployed

REVISION: 2

TEST SUITE: None

...

➜ helm -n mercury ls

NAME                            NAMESPACE     STATUS          CHART           APP VERSION

internal-issue-report-apiv2     mercury       deployed        nginx-9.5.2     1.21.1     

internal-issue-report-app       mercury       deployed        nginx-9.5.0     1.21.1 

Looking good!

> **_INFO:_** Also check out `helm rollback` for undoing a helm rollout/upgrade

**3.**

Now we're asked to install a new release, with a customised values setting. For this we first list all possible value settings for the chart, we can do this via:

xxxxxxxxxx

helm show values bitnami/apache \# will show a long list of all possible value-settings

helm show values bitnami/apache | yq e \# parse yaml and show with colors

Huge list, if we search in it we should find the setting `replicaCount: 1` on top level. This means we can run:

xxxxxxxxxx

➜ helm -n mercury install internal-issue-report-apache bitnami/apache --set replicaCount=2

NAME: internal-issue-report-apache

LAST DEPLOYED: Tue Aug 31 17:57:23 2021

NAMESPACE: mercury

STATUS: deployed

REVISION: 1

TEST SUITE: None

...

If we would also need to set a value on a deeper level, for example `image.debug`, we could run:

xxxxxxxxxx

helm \-n mercury install internal-issue-report-apache bitnami/apache \\

  \--set replicaCount\=2 \\

  \--set image.debug\=true

Install done, let's verify what we did:

xxxxxxxxxx

➜ helm -n mercury ls

NAME                            NAMESPACE     STATUS          CHART           APP VERSION

internal-issue-report-apache    mercury       deployed        apache-8.6.3    2.4.48

...

➜ k -n mercury get deploy internal-issue-report-apache

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE

internal-issue-report-apache   2/2     2            2           96s

We see a healthy deployment with two replicas!

**4.**

By default releases in `pending-upgrade` state aren't listed, but we can show all to find and delete the broken release:

xxxxxxxxxx

➜ helm -n mercury ls -a

NAME                            NAMESPACE     STATUS          CHART           APP VERSION

internal-issue-report-apache    mercury       deployed        apache-8.6.3    2.4.48     

internal-issue-report-apiv2     mercury       deployed        nginx-9.5.2     1.21.1     

internal-issue-report-app       mercury       deployed        nginx-9.5.0     1.21.1     

internal-issue-report-daniel    mercury       pending-install nginx-9.5.0     1.21.1 

➜ helm -n mercury uninstall internal-issue-report-daniel

release "internal-issue-report-daniel" uninstalled

Thank you Helm for making our lifes easier! (Till something breaks)

Question 5 | ServiceAccount, Secret
-----------------------------------

Solve this question on instance: `ssh ckad7326`

Team Neptune has its own _ServiceAccount_ named `neptune-sa-v2` in _Namespace_ `neptune`. A coworker needs the token from the _Secret_ that belongs to that _ServiceAccount_. Write the base64 decoded token to file `/opt/course/5/token` on `ckad7326`.

##### Answer:

> Since K8s 1.24, _Secrets_ won't be created automatically for _ServiceAccounts_ any longer. But it's still possible to create a _Secret_ manually and attach it to a _ServiceAccount_ by setting the correct annotation on the _Secret_. This was done for this task.

xxxxxxxxxx

k \-n neptune get sa \# get overview

k \-n neptune get secrets \# shows all secrets of namespace

k \-n neptune get secrets \-oyaml | grep annotations \-A 1 \# shows secrets with first annotation

If a _Secret_ belongs to a _ServiceAccont_, it'll have the annotation `kubernetes.io/service-account.name`. Here the _Secret_ we're looking for is `neptune-secret-1`.

xxxxxxxxxx

➜ k -n neptune get secret neptune-secret-1 -o yaml

apiVersion: v1

data:

...

  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltNWFaRmRxWkRKMmFHTnZRM0JxV0haT1IxZzFiM3BJY201SlowaEhOV3hUWmt3elFuRmFhVEZhZDJNaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUp1WlhCMGRXNWxJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbTVsY0hSMWJtVXRjMkV0ZGpJdGRHOXJaVzR0Wm5FNU1tb2lMQ0pyZFdKbGNtNWxkR1Z6TG1sdkwzTmxjblpwWTJWaFkyTnZkVzUwTDNObGNuWnBZMlV0WVdOamIzVnVkQzV1WVcxbElqb2libVZ3ZEhWdVpTMXpZUzEyTWlJc0ltdDFZbVZ5Ym1WMFpYTXVhVzh2YzJWeWRtbGpaV0ZqWTI5MWJuUXZjMlZ5ZG1salpTMWhZMk52ZFc1MExuVnBaQ0k2SWpZMlltUmpOak0yTFRKbFl6TXROREpoWkMwNE9HRTFMV0ZoWXpGbFpqWmxPVFpsTlNJc0luTjFZaUk2SW5ONWMzUmxiVHB6WlhKMmFXTmxZV05qYjNWdWREcHVaWEIwZFc1bE9tNWxjSFIxYm1VdGMyRXRkaklpZlEuVllnYm9NNENUZDBwZENKNzh3alV3bXRhbGgtMnZzS2pBTnlQc2gtNmd1RXdPdFdFcTVGYnc1WkhQdHZBZHJMbFB6cE9IRWJBZTRlVU05NUJSR1diWUlkd2p1Tjk1SjBENFJORmtWVXQ0OHR3b2FrUlY3aC1hUHV3c1FYSGhaWnp5NHlpbUZIRzlVZm1zazVZcjRSVmNHNm4xMzd5LUZIMDhLOHpaaklQQXNLRHFOQlF0eGctbFp2d1ZNaTZ2aUlocnJ6QVFzME1CT1Y4Mk9KWUd5Mm8tV1FWYzBVVWFuQ2Y5NFkzZ1QwWVRpcVF2Y3pZTXM2bno5dXQtWGd3aXRyQlk2VGo5QmdQcHJBOWtfajVxRXhfTFVVWlVwUEFpRU43T3pka0pzSThjdHRoMTBseXBJMUFlRnI0M3Q2QUx5clFvQk0zOWFiRGZxM0Zrc1Itb2NfV013

kind: Secret

...

This shows the base64 encoded token. To get the encoded one we could pipe it manually through `base64 -d` or we simply do:

xxxxxxxxxx

➜ k -n neptune describe secret neptune-secret-1

...

Data

\====

token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Im5aZFdqZDJ2aGNvQ3BqWHZOR1g1b3pIcm5JZ0hHNWxTZkwzQnFaaTFad2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJuZXB0dW5lIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im5lcHR1bmUtc2EtdjItdG9rZW4tZnE5MmoiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibmVwdHVuZS1zYS12MiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY2YmRjNjM2LTJlYzMtNDJhZC04OGE1LWFhYzFlZjZlOTZlNSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpuZXB0dW5lOm5lcHR1bmUtc2EtdjIifQ.VYgboM4CTd0pdCJ78wjUwmtalh-2vsKjANyPsh-6guEwOtWEq5Fbw5ZHPtvAdrLlPzpOHEbAe4eUM95BRGWbYIdwjuN95J0D4RNFkVUt48twoakRV7h-aPuwsQXHhZZzy4yimFHG9Ufmsk5Yr4RVcG6n137y-FH08K8zZjIPAsKDqNBQtxg-lZvwVMi6viIhrrzAQs0MBOV82OJYGy2o-WQVc0UUanCf94Y3gT0YTiqQvczYMs6nz9ut-XgwitrBY6Tj9BgPprA9k\_j5qEx\_LUUZUpPAiEN7OzdkJsI8ctth10lypI1AeFr43t6ALyrQoBM39abDfq3FksR-oc\_WMw

ca.crt:     1066 bytes

namespace:  7 bytes

Copy the token (part under `token:`) and paste it using vim.

xxxxxxxxxx

vim /opt/course/5/token

File `/opt/course/5/token` should contain the token:

xxxxxxxxxx

\# /opt/course/5/token

eyJhbGciOiJSUzI1NiIsImtpZCI6Im5aZFdqZDJ2aGNvQ3BqWHZOR1g1b3pIcm5JZ0hHNWxTZkwzQnFaaTFad2MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJuZXB0dW5lIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im5lcHR1bmUtc2EtdjItdG9rZW4tZnE5MmoiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibmVwdHVuZS1zYS12MiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY2YmRjNjM2LTJlYzMtNDJhZC04OGE1LWFhYzFlZjZlOTZlNSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpuZXB0dW5lOm5lcHR1bmUtc2EtdjIifQ.VYgboM4CTd0pdCJ78wjUwmtalh-2vsKjANyPsh-6guEwOtWEq5Fbw5ZHPtvAdrLlPzpOHEbAe4eUM95BRGWbYIdwjuN95J0D4RNFkVUt48twoakRV7h-aPuwsQXHhZZzy4yimFHG9Ufmsk5Yr4RVcG6n137y-FH08K8zZjIPAsKDqNBQtxg-lZvwVMi6viIhrrzAQs0MBOV82OJYGy2o-WQVc0UUanCf94Y3gT0YTiqQvczYMs6nz9ut-XgwitrBY6Tj9BgPprA9k\_j5qEx\_LUUZUpPAiEN7OzdkJsI8ctth10lypI1AeFr43t6ALyrQoBM39abDfq3FksR-oc\_WMw

Question 6 | ReadinessProbe
---------------------------

Solve this question on instance: `ssh ckad5601`

Create a single _Pod_ named `pod6` in _Namespace_ `default` of image `busybox:1.31.0`. The _Pod_ should have a readiness-probe executing `cat /tmp/ready`. It should initially wait 5 and periodically wait 10 seconds. This will set the container ready only if the file `/tmp/ready` exists.

The _Pod_ should run the command `touch /tmp/ready && sleep 1d`, which will create the necessary file to be ready and then idles. Create the _Pod_ and confirm it starts.

##### Answer:

xxxxxxxxxx

k run pod6 \--image\=busybox:1.31.0 \--dry-run\=client \-oyaml \--command \-- sh \-c "touch /tmp/ready && sleep 1d" > 6.yaml

vim 6.yaml

Search for a readiness-probe example on [https://kubernetes.io/docs](https://kubernetes.io/docs), then copy and alter the relevant section for the task:

xxxxxxxxxx

\# 6.yaml

apiVersion: v1

kind: Pod

metadata:

  creationTimestamp: null

  labels:

    run: pod6

  name: pod6

spec:

  containers:

  - command:

    - sh

    - \-c

    - touch /tmp/ready && sleep 1d

    image: busybox:1.31.0

    name: pod6

    resources: {}

    readinessProbe:                             \# add

      exec:                                     \# add

        command:                                \# add

        - sh  \# add

        - \-c  \# add

        - cat /tmp/ready  \# add

      initialDelaySeconds: 5  \# add

      periodSeconds: 10 \# add

  dnsPolicy: ClusterFirst

  restartPolicy: Always

status: {}

Then:

xxxxxxxxxx

k \-f 6.yaml create

Running `k get pod6` we should see the job being created and completed:

xxxxxxxxxx

➜ k get pod pod6

NAME   READY   STATUS              RESTARTS   AGE

pod6   0/1     ContainerCreating   0          2s

➜ k get pod pod6

NAME   READY   STATUS    RESTARTS   AGE

pod6   0/1     Running   0          7s

➜ k get pod pod6

NAME   READY   STATUS    RESTARTS   AGE

pod6   1/1     Running   0          15s

We see that the _Pod_ is finally ready.

Question 7 | Pods, Namespaces
-----------------------------

Solve this question on instance: `ssh ckad7326`

The board of Team Neptune decided to take over control of one e-commerce webserver from Team Saturn. The administrator who once setup this webserver is not part of the organisation any longer. All information you could get was that the e-commerce system is called `my-happy-shop`.

Search for the correct _Pod_ in _Namespace_ `saturn` and move it to _Namespace_ `neptune`. It doesn't matter if you shut it down and spin it up again, it probably hasn't any customers anyways.

##### Answer:

Let's see all those _Pods_:

xxxxxxxxxx

➜ k -n saturn get pod

NAME                READY   STATUS    RESTARTS   AGE

webserver-sat-001   1/1     Running   0          111m

webserver-sat-002   1/1     Running   0          111m

webserver-sat-003   1/1     Running   0          111m

webserver-sat-004   1/1     Running   0          111m

webserver-sat-005   1/1     Running   0          111m

webserver-sat-006   1/1     Running   0          111m

The _Pod_ names don't reveal any information. We assume the _Pod_ we are searching has a _label_ or _annotation_ with the name `my-happy-shop`, so we search for it:

xxxxxxxxxx

k \-n saturn describe pod \# describe all pods, then manually look for it

\# or do some filtering like this

k \-n saturn get pod \-o yaml | grep my-happy-shop \-A10

We see the webserver we're looking for is `webserver-sat-003`

xxxxxxxxxx

k \-n saturn get pod webserver-sat-003 \-o yaml > 7\_webserver-sat-003.yaml \# export

vim 7\_webserver-sat-003.yaml

Change the _Namespace_ to `neptune`, also remove the `status:` section, the token `volume`, the token `volumeMount` and the `nodeName`, else the new _Pod_ won't start. The final file could look as clean like this:

xxxxxxxxxx

\# 7\_webserver-sat-003.yaml

apiVersion: v1

kind: Pod

metadata:

  annotations:

    description: this is the server for the E-Commerce System my-happy-shop

  labels:

    id: webserver-sat-003

  name: webserver-sat-003

  namespace: neptune \# new namespace here

spec:

  containers:

  - image: nginx:1.16.1-alpine

    imagePullPolicy: IfNotPresent

    name: webserver-sat

  restartPolicy: Always

Then we execute:

xxxxxxxxxx

k \-n neptune create \-f 7\_webserver-sat-003.yaml

xxxxxxxxxx

➜ k -n neptune get pod | grep webserver

webserver-sat-003               1/1     Running            0          22s

It seems the server is running in _Namespace_ `neptune`, so we can do:

xxxxxxxxxx

k \-n saturn delete pod webserver-sat-003 \--force \--grace-period\=0

Let's confirm only one is running:

xxxxxxxxxx

➜ k get pod -A | grep webserver-sat-003

neptune        webserver-sat-003         1/1     Running            0          6s

This should list only one pod called `webserver-sat-003` in _Namespace_ `neptune`, status running.

Question 8 | Deployment, Rollouts
---------------------------------

Solve this question on instance: `ssh ckad7326`

There is an existing _Deployment_ named `api-new-c32` in _Namespace_ `neptune`. A developer did make an update to the _Deployment_ but the updated version never came online. Check the _Deployment_ history and find a revision that works, then rollback to it. Could you tell Team Neptune what the error was so it doesn't happen again?

##### Answer:

xxxxxxxxxx

k \-n neptune get deploy \# overview

k \-n neptune rollout \-h

k \-n neptune rollout history \-h

xxxxxxxxxx

➜ k -n neptune rollout history deploy api-new-c32

deployment.extensions/api-new-c32 

REVISION  CHANGE-CAUSE

1         <none>

2         kubectl edit deployment api-new-c32 --namespace=neptune

3         kubectl edit deployment api-new-c32 --namespace=neptune

4         kubectl edit deployment api-new-c32 --namespace=neptune

5         kubectl edit deployment api-new-c32 --namespace=neptune

We see 5 revisions, let's check _Pod_ and _Deployment_ status:

xxxxxxxxxx

➜ k -n neptune get deploy,pod | grep api-new-c32

deployment.extensions/api-new-c32    3/3     1            3           141m

pod/api-new-c32-65d998785d-jtmqq    1/1     Running            0          141m

pod/api-new-c32-686d6f6b65-mj2fp    1/1     Running            0          141m

pod/api-new-c32-6dd45bdb68-2p462    1/1     Running            0          141m

pod/api-new-c32-7d64747c87-zh648    0/1     ImagePullBackOff   0          141m

Let's check the pod for errors:

xxxxxxxxxx

➜ k -n neptune describe pod api-new-c32-7d64747c87-zh648 | grep -i error

  ...  Error: ImagePullBackOff

xxxxxxxxxx

➜ k -n neptune describe pod api-new-c32-7d64747c87-zh648 | grep -i image

    Image:          ngnix:1.16.3

    Image ID:

      Reason:       ImagePullBackOff

  Warning  Failed  4m28s (x616 over 144m)  kubelet, gke-s3ef67020-28c5-45f7--default-pool-248abd4f-s010  Error: ImagePullBackOff

Someone seems to have added a new image with a spelling mistake in the name `ngnix:1.16.3`, that's the reason we can tell Team Neptune!

Now let's revert to the previous version:

xxxxxxxxxx

k \-n neptune rollout undo deploy api-new-c32

Does this one work?

xxxxxxxxxx

➜ k -n neptune get deploy api-new-c32

NAME          READY   UP-TO-DATE   AVAILABLE   AGE

api-new-c32   3/3     3            3           146m

Yes! All up-to-date and available.

Also a fast way to get an overview of the _ReplicaSets_ of a _Deployment_ and their images could be done with:

xxxxxxxxxx

k \-n neptune get rs \-o wide | grep api-new-c32

Question 9 | Pod -> Deployment
------------------------------

Solve this question on instance: `ssh ckad9043`

In _Namespace_ `pluto` there is single _Pod_ named `holy-api`. It has been working okay for a while now but Team Pluto needs it to be more reliable.

Convert the _Pod_ into a _Deployment_ named `holy-api` with 3 replicas and delete the single _Pod_ once done. The raw _Pod_ template file is available at `/opt/course/9/holy-api-pod.yaml`.

In addition, the new _Deployment_ should set `allowPrivilegeEscalation: false` and `privileged: false` for the security context on container level.

Please create the _Deployment_ and save its yaml under `/opt/course/9/holy-api-deployment.yaml` on `ckad9043`.

##### Answer

There are multiple ways to do this, one is to copy an _Deployment_ example from [https://kubernetes.io/docs](https://kubernetes.io/docs) and then merge it with the existing _Pod_ yaml. That's what we will do now:

xxxxxxxxxx

cp /opt/course/9/holy-api-pod.yaml /opt/course/9/holy-api-deployment.yaml \# make a copy!

vim /opt/course/9/holy-api-deployment.yaml

Now copy/use a _Deployment_ example yaml and put the _Pod's_ **metadata:** and **spec:** into the _Deployment's_ **template:** section:

xxxxxxxxxx

\# /opt/course/9/holy-api-deployment.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: holy-api        \# name stays the same

  namespace: pluto \# important

spec:

  replicas: 3 \# 3 replicas

  selector:

    matchLabels:

      id: holy-api  \# set the correct selector

  template:

    \# => from here down its the same as the pods metadata: and spec: sections

    metadata:

      labels:

        id: holy-api

      name: holy-api

    spec:

      containers:

      - env:

        - name: CACHE\_KEY\_1

          value: b&MTCi0=\[T66RXm!jO@

        - name: CACHE\_KEY\_2

          value: PCAILGej5Ld@Q%{Q1=#

        - name: CACHE\_KEY\_3

          value: 2qz-\]2OJlWDSTn\_;RFQ

        image: nginx:1.17.3-alpine

        name: holy-api-container

        securityContext:                   \# add

          allowPrivilegeEscalation: false  \# add

          privileged: false                \# add

        volumeMounts:

        - mountPath: /cache1

          name: cache-volume1

        - mountPath: /cache2

          name: cache-volume2

        - mountPath: /cache3

          name: cache-volume3

      volumes:

      - emptyDir: {}

        name: cache-volume1

      - emptyDir: {}

        name: cache-volume2

      - emptyDir: {}

        name: cache-volume3

To indent multiple lines using `vim` you should set the shiftwidth using `:set shiftwidth=2`. Then mark multiple lines using `Shift v` and the up/down keys.

To then indent the marked lines press `>` or `<` and to repeat the action press `.`

Next create the new _Deployment_:

xxxxxxxxxx

k \-f /opt/course/9/holy-api-deployment.yaml create

and confirm it's running:

xxxxxxxxxx

➜ k -n pluto get pod | grep holy

NAME                        READY   STATUS    RESTARTS   AGE

holy-api                    1/1     Running   0          19m

holy-api-5dbfdb4569-8qr5x   1/1     Running   0          30s

holy-api-5dbfdb4569-b5clh   1/1     Running   0          30s

holy-api-5dbfdb4569-rj2gz   1/1     Running   0          30s

Finally delete the single _Pod_:

xxxxxxxxxx

k \-n pluto delete pod holy-api \--force \--grace-period\=0

xxxxxxxxxx

➜ k -n pluto get pod,deployment | grep holy

pod/holy-api-5dbfdb4569-8qr5x   1/1     Running   0          2m4s

pod/holy-api-5dbfdb4569-b5clh   1/1     Running   0          2m4s

pod/holy-api-5dbfdb4569-rj2gz   1/1     Running   0          2m4s

deployment.extensions/holy-api   3/3     3            3           2m4s

Question 10 | Service, Logs
---------------------------

Solve this question on instance: `ssh ckad9043`

Team Pluto needs a new cluster internal _Service_. Create a ClusterIP _Service_ named `project-plt-6cc-svc` in _Namespace_ `pluto`. This _Service_ should expose a single _Pod_ named `project-plt-6cc-api` of image `nginx:1.17.3-alpine`, create that _Pod_ as well. The _Pod_ should be identified by label `project: plt-6cc-api`. The _Service_ should use tcp port redirection of `3333:80`.

Finally use for example `curl` from a temporary `nginx:alpine` _Pod_ to get the response from the _Service_. Write the response into `/opt/course/10/service_test.html` on `ckad9043`. Also check if the logs of _Pod_ `project-plt-6cc-api` show the request and write those into `/opt/course/10/service_test.log` on `ckad9043`.

##### Answer

xxxxxxxxxx

k \-n pluto run project-plt-6cc-api \--image\=nginx:1.17.3-alpine \--labels project\=plt-6cc-api

This will create the requested _Pod_. In yaml it would look like this:

xxxxxxxxxx

apiVersion: v1

kind: Pod

metadata:

  creationTimestamp: null

  labels:

    project: plt-6cc-api

  name: project-plt-6cc-api

spec:

  containers:

  - image: nginx:1.17.3-alpine

    name: project-plt-6cc-api

    resources: {}

  dnsPolicy: ClusterFirst

  restartPolicy: Always

status: {}

Next we create the service:

xxxxxxxxxx

k \-n pluto expose pod \-h \# help

k \-n pluto expose pod project-plt-6cc-api \--name project-plt-6cc-svc \--port 3333 \--target-port 80

Expose will create a yaml where everything is already set for our case and no need to change anything:

xxxxxxxxxx

apiVersion: v1

kind: Service

metadata:

  creationTimestamp: null

  labels:

    project: plt-6cc-api

  name: project-plt-6cc-svc \# good

  namespace: pluto  \# great

spec:

  ports:

  - port: 3333  \# awesome

    protocol: TCP

    targetPort: 80  \# nice

  selector:

    project: plt-6cc-api \# beautiful

status:

  loadBalancer: {}

We could also use `create service` but then we would need to change the yaml afterwards:

xxxxxxxxxx

k \-n pluto create service \-h \# help

k \-n pluto create service clusterip \-h #help

k \-n pluto create service clusterip project-plt-6cc-svc \--tcp 3333:80 \--dry-run\=client \-oyaml

\# now we would need to set the correct selector labels

Check the _Service_ is running:

xxxxxxxxxx

➜ k -n pluto get pod,svc | grep 6cc

pod/project-plt-6cc-api         1/1     Running   0          9m42s

service/project-plt-6cc-svc   ClusterIP   10.31.241.234   <none>        3333/TCP   2m24s

Does the _Service_ has one _Endpoint_?

xxxxxxxxxx

➜ k -n pluto describe svc project-plt-6cc-svc

Name:              project-plt-6cc-svc

Namespace:         pluto

Labels:            project=plt-6cc-api

Annotations:       <none>

Selector:          project=plt-6cc-api

Type:              ClusterIP

IP:                10.3.244.240

Port:              <unset>  3333/TCP

TargetPort:        80/TCP

Endpoints:         10.28.2.32:80 

Session Affinity:  None

Events:            <none>

Or even shorter:

xxxxxxxxxx

➜ k -n pluto get ep

NAME                  ENDPOINTS       AGE

project-plt-6cc-svc   10.28.2.32:80   84m

Yes, endpoint there! Finally we check the connection using a temporary _Pod_:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm --image=nginx:alpine -i -- curl http://project-plt-6cc-svc.pluto:3333

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100   612  100   612    0     0  32210      0 --:--:-- --:--:-- --:--:-- 32210

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

<style>

    body {

        width: 35em;

        margin: 0 auto;

        font-family: Tahoma, Verdana, Arial, sans-serif;

    }

</style>

</head>

<body>

<h1>Welcome to nginx!</h1>

...

Great! Notice that we use the Kubernetes _Namespace_ dns resolving (`project-plt-6cc-svc.pluto`) here. We could only use the _Service_ name if we would also spin up the temporary _Pod_ in _Namespace_ `pluto` .

And now really finally copy or pipe the html content into `/opt/course/10/service_test.html`.

xxxxxxxxxx

\# /opt/course/10/service\_test.html

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

<style>

    body {

        width: 35em;

        margin: 0 auto;

        font-family: Tahoma, Verdana, Arial, sans-serif;

    }

...

Also the requested logs:

xxxxxxxxxx

k \-n pluto logs project-plt-6cc-api > /opt/course/10/service\_test.log

xxxxxxxxxx

\# /opt/course/10/service\_test.log

10.44.0.0 - - \[22/Jan/2021:23:19:55 +0000\] "GET / HTTP/1.1" 200 612 "-" "curl/7.69.1" "-"

Question 11 | Working with Containers
-------------------------------------

Solve this question on instance: `ssh ckad9043`

During the last monthly meeting you mentioned your strong expertise in container technology. Now the Build&Release team of department Sun is in need of your insight knowledge. There are files to build a container image located at `/opt/course/11/image`. The container will run a Golang application which outputs information to stdout. You're asked to perform the following tasks:

> **_NOTE:_** Make sure to run all commands as user `candidate`, for docker use `sudo docker`

1.  Change the Dockerfile. The value of the environment variable `SUN_CIPHER_ID` should be set to the hardcoded value `5b9c1065-e39d-4a43-a04a-e59bcea3e03f`
    
2.  Build the image using Docker, named `registry.killer.sh:5000/sun-cipher`, tagged as `latest` and `v1-docker`, push these to the registry
    
3.  Build the image using Podman, named `registry.killer.sh:5000/sun-cipher`, tagged as `v1-podman`, push it to the registry
    
4.  Run a container using Podman, which keeps running in the background, named `sun-cipher` using image `registry.killer.sh:5000/sun-cipher:v1-podman`. Run the container from `candidate@ckad9043` and not `root@ckad9043`
    
5.  Write the logs your container `sun-cipher` produced into `/opt/course/11/logs`. Then write a list of all running Podman containers into `/opt/course/11/containers` on `ckad9043`
    

##### Answer

_Dockerfile_: list of commands from which an _Image_ can be build

_Image_: binary file which includes all data/requirements to be run as a _Container_

_Container_: running instance of an _Image_

_Registry_: place where we can push/pull _Images_ to/from

**1.**

First we need to change the Dockerfile to:

xxxxxxxxxx

\# build container stage 1

FROM docker.io/library/golang:1.15.15-alpine3.14

WORKDIR /src

COPY . .

RUN CGO\_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o bin/app .

\# app container stage 2

FROM docker.io/library/alpine:3.12.4

COPY --from=0 /src/bin/app app

\# CHANGE NEXT LINE

ENV SUN\_CIPHER\_ID=5b9c1065-e39d-4a43-a04a-e59bcea3e03f

CMD \["./app"\]

**2.**

Then we build the image using Docker:

xxxxxxxxxx

➜ cd /opt/course/11/image

➜ sudo docker build -t registry.killer.sh:5000/sun-cipher:latest -t registry.killer.sh:5000/sun-cipher:v1-docker .

...

Successfully built 409fde3c5bf9

Successfully tagged registry.killer.sh:5000/sun-cipher:latest

Successfully tagged registry.killer.sh:5000/sun-cipher:v1-docker

➜ sudo docker image ls

REPOSITORY                           TAG         IMAGE ID       CREATED              SIZE

registry.killer.sh:5000/sun-cipher   latest      409fde3c5bf9   24 seconds ago       7.76MB

registry.killer.sh:5000/sun-cipher   v1-docker   409fde3c5bf9   24 seconds ago       7.76MB

...

➜ sudo docker push registry.killer.sh:5000/sun-cipher:latest

The push refers to repository \[registry.killer.sh:5000/sun-cipher\]

c947fb5eba52: Pushed 

33e8713114f8: Pushed 

latest: digest: sha256:d216b4136a5b232b738698e826e7d12fccba9921d163b63777be23572250f23d size: 739

➜ sudo docker push registry.killer.sh:5000/sun-cipher:v1-docker

The push refers to repository \[registry.killer.sh:5000/sun-cipher\]

c947fb5eba52: Layer already exists 

33e8713114f8: Layer already exists 

v1-docker: digest: sha256:d216b4136a5b232b738698e826e7d12fccba9921d163b63777be23572250f23d size: 739

There we go, built and pushed.

**3.**

Next we build the image using Podman. Here it's only required to create one tag. The usage of Podman is very similar (for most cases even identical) to Docker:

xxxxxxxxxx

➜ cd /opt/course/11/image

➜ podman build -t registry.killer.sh:5000/sun-cipher:v1-podman .

...

\--> 38adc53bd92

Successfully tagged registry.killer.sh:5000/sun-cipher:v1-podman

38adc53bd92881d91981c4b537f4f1b64f8de1de1b32eacc8479883170cee537

➜ podman image ls

REPOSITORY                          TAG         IMAGE ID      CREATED        SIZE

registry.killer.sh:5000/sun-cipher  v1-podman   38adc53bd928  2 minutes ago  8.03 MB

...

➜ podman push registry.killer.sh:5000/sun-cipher:v1-podman

Getting image source signatures

Copying blob 4d0d60db9eb6 done  

Copying blob 33e8713114f8 done  

Copying config bfa1a225f8 done  

Writing manifest to image destination

Storing signatures

Built and pushed using Podman.

**4.**

We'll create a container from the perviously created image, using Podman, which keeps running in the background:

xxxxxxxxxx

➜ podman run -d --name sun-cipher registry.killer.sh:5000/sun-cipher:v1-podman

f8199cba792f9fd2d1bd4decc9b7a9c0acfb975d95eda35f5f583c9efbf95589

**5.**

Finally we need to collect some information into files:

xxxxxxxxxx

➜ podman ps

CONTAINER ID  IMAGE                                         COMMAND     ...       

f8199cba792f  registry.killer.sh:5000/sun-cipher:v1-podman  ./app       ...

➜ podman ps > /opt/course/11/containers

➜ podman logs sun-cipher

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 8081

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 7887

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 1847

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 4059

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 2081

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 1318

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 4425

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 2540

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 456

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 3300

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 694

2077/03/13 06:50:34 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 8511

2077/03/13 06:50:44 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 8162

2077/03/13 06:50:54 random number for 5b9c1065-e39d-4a43-a04a-e59bcea3e03f is 5089

➜ podman logs sun-cipher > /opt/course/11/logs

This is looking not too bad at all. Our container skills are back in town!

Question 12 | Storage, PV, PVC, Pod volume
------------------------------------------

Solve this question on instance: `ssh ckad5601`

Create a new _PersistentVolume_ named `earth-project-earthflower-pv`. It should have a capacity of _2Gi_, accessMode _ReadWriteOnce_, hostPath `/Volumes/Data` and no storageClassName defined.

Next create a new _PersistentVolumeClaim_ in _Namespace_ `earth` named `earth-project-earthflower-pvc` . It should request _2Gi_ storage, accessMode _ReadWriteOnce_ and should not define a storageClassName. The _PVC_ should bound to the _PV_ correctly.

Finally create a new _Deployment_ `project-earthflower` in _Namespace_ `earth` which mounts that volume at `/tmp/project-data`. The _Pods_ of that _Deployment_ should be of image `httpd:2.4.41-alpine`.

##### Answer

xxxxxxxxxx

vim 12\_pv.yaml

Find an example from [https://kubernetes.io/docs](https://kubernetes.io/docs) and alter it:

xxxxxxxxxx

\# 12\_pv.yaml

kind: PersistentVolume

apiVersion: v1

metadata:

 name: earth-project-earthflower-pv

spec:

 capacity:

  storage: 2Gi

 accessModes:

  - ReadWriteOnce

 hostPath:

  path: "/Volumes/Data"

Then create it:

xxxxxxxxxx

k \-f 12\_pv.yaml create

Next the _PersistentVolumeClaim_:

xxxxxxxxxx

vim 12\_pvc.yaml

Find an example from [https://kubernetes.io/docs](https://kubernetes.io/docs) and alter it:

xxxxxxxxxx

\# 12\_pvc.yaml

kind: PersistentVolumeClaim

apiVersion: v1

metadata:

  name: earth-project-earthflower-pvc

  namespace: earth

spec:

  accessModes:

    - ReadWriteOnce

  resources:

    requests:

     storage: 2Gi

Then create:

xxxxxxxxxx

k \-f 12\_pvc.yaml create

And check that both have the status Bound:

xxxxxxxxxx

➜ k -n earth get pv,pvc

NAME                                 CAPACITY   ACCESS MODES   ...  STATUS   CLAIM 

persistentvolume/...earthflower-pv   2Gi        RWO            ...  Bound    ...er-pvc

NAME                                       STATUS   VOLUME                         CAPACITY

persistentvolumeclaim/...earthflower-pvc   Bound    earth-project-earthflower-pv   2Gi

Next we create a _Deployment_ and mount that volume:

xxxxxxxxxx

k \-n earth create deploy project-earthflower \--image\=httpd:2.4.41-alpine \--dry-run\=client \-oyaml > 12\_dep.yaml

vim 12\_dep.yaml

Alter the yaml to mount the volume:

xxxxxxxxxx

\# 12\_dep.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  creationTimestamp: null

  labels:

    app: project-earthflower

  name: project-earthflower

  namespace: earth

spec:

  replicas: 1

  selector:

    matchLabels:

      app: project-earthflower

  strategy: {}

  template:

    metadata:

      creationTimestamp: null

      labels:

        app: project-earthflower

    spec:

      volumes:                                      \# add

      - name: data        \# add

        persistentVolumeClaim:                      \# add

          claimName: earth-project-earthflower-pvc  \# add

      containers:

      - image: httpd:2.4.41-alpine

        name: container

        volumeMounts:                               \# add

        - name: data        \# add

          mountPath: /tmp/project-data        \# add

xxxxxxxxxx

k \-f 12\_dep.yaml create

We can confirm it's mounting correctly:

xxxxxxxxxx

➜ k -n earth describe pod project-earthflower-d6887f7c5-pn5wv | grep -A2 Mounts:   

    Mounts:

      /tmp/project-data from data (rw) # there it is

      /var/run/secrets/kubernetes.io/serviceaccount from default-token-n2sjj (ro)

Question 13 | Storage, StorageClass, PVC
----------------------------------------

Solve this question on instance: `ssh ckad9043`

Team Moonpie, which has the _Namespace_ `moon`, needs more storage. Create a new _PersistentVolumeClaim_ named `moon-pvc-126` in that namespace. This claim should use a new _StorageClass_ `moon-retain` with the _provisioner_ set to `moon-retainer` and the _reclaimPolicy_ set to _Retain_. The claim should request storage of _3Gi_, an _accessMode_ of _ReadWriteOnce_ and should use the new _StorageClass_.

The provisioner `moon-retainer` will be created by another team, so it's expected that the _PVC_ will not boot yet. Confirm this by writing the log message from the _PVC_ into file `/opt/course/13/pvc-126-reason` on `ckad9043`.

##### Answer

xxxxxxxxxx

vim 13\_sc.yaml

Head to [https://kubernetes.io/docs](https://kubernetes.io/docs), search for "storageclass" and alter the example code to this:

xxxxxxxxxx

\# 13\_sc.yaml

apiVersion: storage.k8s.io/v1

kind: StorageClass

metadata:

  name: moon-retain

provisioner: moon-retainer

reclaimPolicy: Retain

xxxxxxxxxx

k create \-f 13\_sc.yaml

Now the same for the _PersistentVolumeClaim_, head to the docs, copy an example and transform it into:

xxxxxxxxxx

vim 13\_pvc.yaml

xxxxxxxxxx

\# 13\_pvc.yaml

apiVersion: v1

kind: PersistentVolumeClaim

metadata:

  name: moon-pvc-126  \# name as requested

  namespace: moon \# important

spec:

  accessModes:

    - ReadWriteOnce \# RWO

  resources:

    requests:

      storage: 3Gi  \# size

  storageClassName: moon-retain \# uses our new storage class

xxxxxxxxxx

k \-f 13\_pvc.yaml create

Next we check the status of the _PVC_ :

xxxxxxxxxx

➜ k -n moon get pvc

NAME       STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE

moon-pvc-126   Pending                                      moon-retain    2m57s

xxxxxxxxxx

➜ k -n moon describe pvc moon-pvc-126

Name:          moon-pvc-126

...

Status:        Pending

...

Events:

...

waiting for a volume to be created, either by external provisioner "moon-retainer" or manually created by system administrator

This confirms that the _PVC_ waits for the provisioner `moon-retainer` to be created. Finally we copy or write the event message into the requested location:

xxxxxxxxxx

\# /opt/course/13/pvc-126-reason

waiting for a volume to be created, either by external provisioner "moon-retainer" or manually created by system administrator

Question 14 | Secret, Secret-Volume, Secret-Env
-----------------------------------------------

Solve this question on instance: `ssh ckad9043`

You need to make changes on an existing _Pod_ in _Namespace_ `moon` called `secret-handler`. Create a new _Secret_ `secret1` which contains `user=test` and `pass=pwd`. The _Secret_'s content should be available in _Pod_ `secret-handler` as environment variables `SECRET1_USER` and `SECRET1_PASS`. The yaml for _Pod_ `secret-handler` is available at `/opt/course/14/secret-handler.yaml`.

There is existing yaml for another _Secret_ at `/opt/course/14/secret2.yaml`, create this _Secret_ and mount it inside the same _Pod_ at `/tmp/secret2`. Your changes should be saved under `/opt/course/14/secret-handler-new.yaml` on `ckad9043`. Both _Secrets_ should only be available in _Namespace_ `moon`.

##### Answer

xxxxxxxxxx

k \-n moon get pod \# show pods

k \-n moon create secret \-h \# help

k \-n moon create secret generic \-h \# help

k \-n moon create secret generic secret1 \--from-literal user\=test \--from-literal pass\=pwd

The last command would generate this yaml:

xxxxxxxxxx

apiVersion: v1

data:

  pass: cHdk

  user: dGVzdA==

kind: Secret

metadata:

  creationTimestamp: null

  name: secret1

  namespace: moon

Next we create the second _Secret_ from the given location, making sure it'll be created in _Namespace_ `moon`:

xxxxxxxxxx

k \-n moon \-f /opt/course/14/secret2.yaml create

xxxxxxxxxx

➜ k -n moon get secret

NAME                  TYPE                                  DATA   AGE

default-token-rvzcf   kubernetes.io/service-account-token   3      66m

secret1               Opaque                                2      4m3s

secret2               Opaque                                1      8s

We will now edit the _Pod_ yaml:

xxxxxxxxxx

cp /opt/course/14/secret-handler.yaml /opt/course/14/secret-handler-new.yaml

vim /opt/course/14/secret-handler-new.yaml

Add the following to the yaml:

xxxxxxxxxx

\# /opt/course/14/secret-handler-new.yaml

apiVersion: v1

kind: Pod

metadata:

  labels:

    id: secret-handler

    uuid: 1428721e-8d1c-4c09-b5d6-afd79200c56a

    red\_ident: 9cf7a7c0-fdb2-4c35-9c13-c2a0bb52b4a9

    type: automatic

  name: secret-handler

  namespace: moon

spec:

  volumes:

  - name: cache-volume1

    emptyDir: {}

  - name: cache-volume2

    emptyDir: {}

  - name: cache-volume3

    emptyDir: {}

  - name: secret2-volume  \# add

    secret:                           \# add

      secretName: secret2 \# add

  containers:

  - name: secret-handler

    image: bash:5.0.11

    args: \['bash', '-c', 'sleep 2d'\]

    volumeMounts:

    - mountPath: /cache1

      name: cache-volume1

    - mountPath: /cache2

      name: cache-volume2

    - mountPath: /cache3

      name: cache-volume3

    - name: secret2-volume  \# add

      mountPath: /tmp/secret2 \# add

    env:

    - name: SECRET\_KEY\_1

      value: ">8$kH#kj..i8}HImQd{"

    - name: SECRET\_KEY\_2

      value: "IO=a4L/XkRdvN8jM=Y+"

    - name: SECRET\_KEY\_3

      value: "-7PA0\_Z\]>{pwa43r)\_\_"

    - name: SECRET1\_USER  \# add

      valueFrom:                      \# add

        secretKeyRef:                 \# add

          name: secret1 \# add

          key: user \# add

    - name: SECRET1\_PASS  \# add

      valueFrom:                      \# add

        secretKeyRef:                 \# add

          name: secret1 \# add

          key: pass \# add

There is also the possibility to import all keys from a _Secret_ as env variables at once, though the env variable names will then be the same as in the _Secret_, which doesn't work for the requirements here:

xxxxxxxxxx

  containers:

  - name: secret-handler

...

    envFrom:

    - secretRef:        \# also works for configMapRef

        name: secret1

Then we apply the changes:

xxxxxxxxxx

k \-f /opt/course/14/secret-handler.yaml delete \--force \--grace-period\=0

k \-f /opt/course/14/secret-handler-new.yaml create

Instead of running delete and create we can also use recreate:

xxxxxxxxxx

k \-f /opt/course/14/secret-handler-new.yaml replace \--force \--grace-period\=0

It was not requested directly, but you should always confirm it's working:

xxxxxxxxxx

➜ k -n moon exec secret-handler -- env | grep SECRET1

SECRET1\_USER=test

SECRET1\_PASS=pwd

➜ k -n moon exec secret-handler -- find /tmp/secret2 

/tmp/secret2

/tmp/secret2/..data

/tmp/secret2/key

/tmp/secret2/..2019\_09\_11\_09\_03\_08.147048594

/tmp/secret2/..2019\_09\_11\_09\_03\_08.147048594/key

➜ k -n moon exec secret-handler -- cat /tmp/secret2/key

12345678

Question 15 | ConfigMap, Configmap-Volume
-----------------------------------------

Solve this question on instance: `ssh ckad9043`

Team Moonpie has a nginx server _Deployment_ called `web-moon` in _Namespace_ `moon`. Someone started configuring it but it was never completed. To complete please create a _ConfigMap_ called `configmap-web-moon-html` containing the content of file `/opt/course/15/web-moon.html` under the data key-name `index.html`.

The _Deployment_ `web-moon` is already configured to work with this _ConfigMap_ and serve its content. Test the nginx configuration for example using `curl` from a temporary `nginx:alpine` _Pod_.

##### Answer

Let's check the existing _Pods_:

xxxxxxxxxx

➜ k -n moon get pod

NAME                        READY   STATUS              RESTARTS   AGE

secret-handler              1/1     Running             0          55m

web-moon-847496c686-2rzj4   0/1     ContainerCreating   0          33s

web-moon-847496c686-9nwwj   0/1     ContainerCreating   0          33s

web-moon-847496c686-cxdbx   0/1     ContainerCreating   0          33s

web-moon-847496c686-hvqlw   0/1     ContainerCreating   0          33s

web-moon-847496c686-tj7ct   0/1     ContainerCreating   0          33s

xxxxxxxxxx

➜ k -n moon describe pod web-moon-847496c686-2rzj4

...

Warning  FailedMount  31s (x7 over 63s)  kubelet, gke-test-default-pool-ce83a51a-p6s4  MountVolume.SetUp failed for volume "html-volume" : configmaps "configmap-web-moon-html" not found

Good so far, now let's create the missing _ConfigMap_:

xxxxxxxxxx

k \-n moon create configmap \-h \# help

k \-n moon create configmap configmap-web-moon-html \--from-file\=index.html\=/opt/course/15/web-moon.html \# important to set the index.html key

This should create a _ConfigMap_ with yaml like:

xxxxxxxxxx

apiVersion: v1

data:

  index.html: |     \# notice the key index.html, this will be the filename when mounted

    <!DOCTYPE html>

    <html lang="en">

    <head>

        <meta charset="UTF-8">

        <title>Web Moon Webpage</title>

    </head>

    <body>

    This is some great content.

    </body>

    </html>

kind: ConfigMap

metadata:

  creationTimestamp: null

  name: configmap-web-moon-html

  namespace: moon

After waiting a bit or deleting/recreating (`k -n moon rollout restart deploy web-moon`) the _Pods_ we should see:

xxxxxxxxxx

➜ k -n moon get pod

NAME                        READY   STATUS    RESTARTS   AGE

secret-handler              1/1     Running   0          59m

web-moon-847496c686-2rzj4   1/1     Running   0          4m28s

web-moon-847496c686-9nwwj   1/1     Running   0          4m28s

web-moon-847496c686-cxdbx   1/1     Running   0          4m28s

web-moon-847496c686-hvqlw   1/1     Running   0          4m28s

web-moon-847496c686-tj7ct   1/1     Running   0          4m28s

Looking much better. Finally we check if the nginx returns the correct content:

xxxxxxxxxx

k \-n moon get pod \-o wide \# get pod cluster IPs

Then use one IP to test the configuration:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl 10.44.0.78

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100   161  100   161    0     0  80500      0 --:--:-- --:--:-- --:--:--  157k

<!DOCTYPE html>

<html lang="en">

<head>

    <meta charset="UTF-8">

    <title>Web Moon Webpage</title>

</head>

<body>

This is some great content.

</body>

For debugging or further checks we could find out more about the _Pods_ volume mounts:

xxxxxxxxxx

➜ k -n moon describe pod web-moon-c77655cc-dc8v4 | grep -A2 Mounts:

    Mounts:

      /usr/share/nginx/html from html-volume (rw)

      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rvzcf (ro)

And check the mounted folder content:

xxxxxxxxxx

➜ k -n moon exec web-moon-c77655cc-dc8v4 find /usr/share/nginx/html

/usr/share/nginx/html

/usr/share/nginx/html/..2019\_09\_11\_10\_05\_56.336284411

/usr/share/nginx/html/..2019\_09\_11\_10\_05\_56.336284411/index.html

/usr/share/nginx/html/..data

/usr/share/nginx/html/index.html

Here it was important that the file will have the name `index.html` and not the original one `web-moon.html` which is controlled through the _ConfigMap_ data key.

Question 16 | Logging sidecar
-----------------------------

Solve this question on instance: `ssh ckad7326`

The Tech Lead of Mercury2D decided it's time for more logging, to finally fight all these missing data incidents. There is an existing container named `cleaner-con` in _Deployment_ `cleaner` in _Namespace_ `mercury`. This container mounts a volume and writes logs into a file called `cleaner.log`.

The yaml for the existing _Deployment_ is available at `/opt/course/16/cleaner.yaml`. Persist your changes at `/opt/course/16/cleaner-new.yaml` on `ckad7326` but also make sure the _Deployment_ is running.

Create a sidecar container named `logger-con`, image `busybox:1.31.0` , which mounts the same volume and writes the content of `cleaner.log` to stdout, you can use the `tail -f` command for this. This way it can be picked up by `kubectl logs`.

Check if the logs of the new container reveal something about the missing data incidents.

##### Answer

Sidecar containers in K8s are `initContainers` with `restartPolicy: Always`. Search for "Sidecar Containers" in the K8s Docs to familiarise yourself if necessary.

xxxxxxxxxx

cp /opt/course/16/cleaner.yaml /opt/course/16/cleaner-new.yaml

vim /opt/course/16/cleaner-new.yaml

Add a sidecar container which outputs the log file to stdout:

x

\# /opt/course/16/cleaner-new.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  creationTimestamp: null

  name: cleaner

  namespace: mercury

spec:

  replicas: 2

  selector:

    matchLabels:

      id: cleaner

  template:

    metadata:

      labels:

        id: cleaner

    spec:

      volumes:

      - name: logs

        emptyDir: {}

      initContainers:

      - name: init

        image: bash:5.0.11

        command: \['bash', '-c', 'echo init > /var/log/cleaner/cleaner.log'\]

        volumeMounts:

        - name: logs

          mountPath: /var/log/cleaner

      - name: logger-con                                                \# add

        image: busybox:1.31.0                                           \# add

        restartPolicy: Always                                           \# add

        command: \["sh", "-c", "tail -f /var/log/cleaner/cleaner.log"\]   \# add

        volumeMounts:                                                   \# add

        - name: logs                                                    \# add

          mountPath: /var/log/cleaner                                   \# add

      containers:

      - name: cleaner-con

        image: bash:5.0.11

        args: \['bash', '-c', 'while true; do echo \`date\`: "remove random file" >> /var/log/cleaner/cleaner.log; sleep 1; done'\]

        volumeMounts:

        - name: logs

          mountPath: /var/log/cleaner

In earlier K8s versions it was necessary to define sidecar containers as additional application containers under `containers:` like this:

x

\# LEGACY example of defining sidecar containers in earlier K8s versions

apiVersion: apps/v1

kind: Deployment

metadata:

  creationTimestamp: null

  name: cleaner

  namespace: mercury

spec:

...

  template:

...

    spec:

...

      initContainers:

      - name: init

        image: bash:5.0.11

...

      containers:

      - name: cleaner-con

        image: bash:5.0.11

...

      - name: logger-con                                                \# LEGACY example

        image: busybox:1.31.0                                           \# LEGACY example

        command: \["sh", "-c", "tail -f /var/log/cleaner/cleaner.log"\]   \# LEGACY example

        volumeMounts:                                                   \# LEGACY example

        - name: logs                                                    \# LEGACY example

          mountPath: /var/log/cleaner                                   \# LEGACY example

Then apply the changes and check the logs of the sidecar:

xxxxxxxxxx

k \-f /opt/course/16/cleaner-new.yaml apply

This will cause a deployment rollout of which we can get more details:

xxxxxxxxxx

k \-n mercury rollout history deploy cleaner

k \-n mercury rollout history deploy cleaner \--revision 1

k \-n mercury rollout history deploy cleaner \--revision 2

Check _Pod_ statuses:

​x

➜ k -n mercury get pod

NAME                       READY   STATUS        RESTARTS   AGE

cleaner-86b7758668-9pw6t   2/2     Running       0          6s

cleaner-86b7758668-qgh4v   0/2     Init:0/1      0          1s

➜ k -n mercury get pod

NAME                       READY   STATUS        RESTARTS   AGE

cleaner-86b7758668-9pw6t   2/2     Running       0          14s

cleaner-86b7758668-qgh4v   2/2     Running       0          9s

Finally check the logs of the logging sidecar container:

xxxxxxxxxx

➜ k -n mercury logs cleaner-576967576c-cqtgx -c logger-con

init

Wed Sep 11 10:45:44 UTC 2099: remove random file

Wed Sep 11 10:45:45 UTC 2099: remove random file

...

Mystery solved, something is removing files at random ;) It's important to understand how containers can communicate with each other using volumes.

Question 17 | InitContainer
---------------------------

Solve this question on instance: `ssh ckad5601`

Last lunch you told your coworker from department Mars Inc how amazing _InitContainer_s are. Now he would like to see one in action. There is a _Deployment_ yaml at `/opt/course/17/test-init-container.yaml`. This _Deployment_ spins up a single _Pod_ of image `nginx:1.17.3-alpine` and serves files from a mounted volume, which is empty right now.

Create an _InitContainer_ named `init-con` which also mounts that volume and creates a file `index.html` with content `check this out!` in the root of the mounted volume. For this test we ignore that it doesn't contain valid html.

The _InitContainer_ should be using image `busybox:1.31.0`. Test your implementation for example using `curl` from a temporary `nginx:alpine` _Pod_.

##### Answer

xxxxxxxxxx

cp /opt/course/17/test-init-container.yaml ~/17\_test-init-container.yaml

vim 17\_test-init-container.yaml

Add the _InitContainer_:

xxxxxxxxxx

\# 17\_test-init-container.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: test-init-container

  namespace: mars

spec:

  replicas: 1

  selector:

    matchLabels:

      id: test-init-container

  template:

    metadata:

      labels:

        id: test-init-container

    spec:

      volumes:

      - name: web-content

        emptyDir: {}

      initContainers:                 \# initContainer start

      - name: init-con

        image: busybox:1.31.0

        command: \['sh', '-c', 'echo "check this out!" > /tmp/web-content/index.html'\]

        volumeMounts:

        - name: web-content

          mountPath: /tmp/web-content \# initContainer end

      containers:

      - image: nginx:1.17.3-alpine

        name: nginx

        volumeMounts:

        - name: web-content

          mountPath: /usr/share/nginx/html

        ports:

        - containerPort: 80

Then we create the _Deployment_:

xxxxxxxxxx

k \-f 17\_test-init-container.yaml create

Finally we test the configuration:

xxxxxxxxxx

k \-n mars get pod \-o wide \# to get the cluster IP

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl 10.0.0.67

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

check this out!

Beautiful.

Question 18 | Service misconfiguration
--------------------------------------

Solve this question on instance: `ssh ckad5601`

There seems to be an issue in _Namespace_ `mars` where the ClusterIP service `manager-api-svc` should make the _Pods_ of _Deployment_ `manager-api-deployment` available inside the cluster.

You can test this with `curl manager-api-svc.mars:4444` from a temporary `nginx:alpine` _Pod_. Check for the misconfiguration and apply a fix.

##### Answer

First let's get an overview:

xxxxxxxxxx

➜ k -n mars get all

NAME                                         READY   STATUS    RESTARTS   AGE

pod/manager-api-deployment-dbcc6657d-bg2hh   1/1     Running   0          98m

pod/manager-api-deployment-dbcc6657d-f5fv4   1/1     Running   0          98m

pod/manager-api-deployment-dbcc6657d-httjv   1/1     Running   0          98m

pod/manager-api-deployment-dbcc6657d-k98xn   1/1     Running   0          98m

pod/test-init-container-5db7c99857-htx6b     1/1     Running   0          2m19s

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE

service/manager-api-svc   ClusterIP   10.15.241.159   <none>        4444/TCP   99m

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE

deployment.apps/manager-api-deployment   4/4     4            4           98m

deployment.apps/test-init-container      1/1     1            1           2m19s

...

Everything seems to be running, but we can't seem to get a connection:

xxxxxxxxxx

➜ k -n mars run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 manager-api-svc:4444

If you don't see a command prompt, try pressing enter.

  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0

curl: (28) Connection timed out after 1000 milliseconds

pod "tmp" deleted

pod mars/tmp terminated (Error)

Ok, let's try to connect to one pod directly:

xxxxxxxxxx

k \-n mars get pod \-o wide \# get cluster IP

xxxxxxxxxx

➜ k -n mars run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.0.1.14

 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

...

The _Pods_ itself seem to work. Let's investigate the _Service_ a bit:

xxxxxxxxxx

➜ k -n mars describe service manager-api-svc

Name:              manager-api-svc

Namespace:         mars

Labels:            app=manager-api-svc

...

Endpoints:         <none>

...

Endpoint inspection is also possible using:

xxxxxxxxxx

k \-n mars get ep

No endpoints - No good. We check the _Service_ yaml:

xxxxxxxxxx

k \-n mars edit service manager-api-svc

xxxxxxxxxx

\# k -n mars edit service manager-api-svc

apiVersion: v1

kind: Service

metadata:

...

  labels:

    app: manager-api-svc

  name: manager-api-svc

  namespace: mars

...

spec:

  clusterIP: 10.3.244.121

  ports:

  - name: 4444-80

    port: 4444

    protocol: TCP

    targetPort: 80

  selector:

    #id: manager-api-deployment # wrong selector, needs to point to pod!

    id: manager-api-pod

  sessionAffinity: None

  type: ClusterIP

Though _Pods_ are usually never created without a _Deployment_ or _ReplicaSet_, _Services_ always select for _Pods_ directly. This gives great flexibility because _Pods_ could be created through various customized ways. After saving the new selector we check the _Service_ again for endpoints:

xxxxxxxxxx

➜ k -n mars get ep

NAME              ENDPOINTS                                               AGE

manager-api-svc   10.0.0.30:80,10.0.1.30:80,10.0.1.31:80 + 1 more...      41m

Endpoints - Good! Now we try connecting again:

xxxxxxxxxx

➜ k -n mars run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 manager-api-svc:4444

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100   612  100   612    0     0    99k      0 --:--:-- --:--:-- --:--:--   99k

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

...

And we fixed it. Good to know is how to be able to use Kubernetes DNS resolution from a different _Namespace_. Not necessary, but we could spin up the temporary _Pod_ in default _Namespace_:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 manager-api-svc:4444

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0curl: (6) Could not resolve host: manager-api-svc

pod "tmp" deleted

pod default/tmp terminated (Error)

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 manager-api-svc.mars:4444

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100   612  100   612    0     0  68000      0 --:--:-- --:--:-- --:--:-- 68000

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

Short `manager-api-svc.mars` or long `manager-api-svc.mars.svc.cluster.local` work.

Question 19 | Service ClusterIP->NodePort
-----------------------------------------

Solve this question on instance: `ssh ckad5601`

In _Namespace_ `jupiter` you'll find an apache _Deployment_ (with one replica) named `jupiter-crew-deploy` and a ClusterIP _Service_ called `jupiter-crew-svc` which exposes it. Change this service to a NodePort one to make it available on all nodes on port 30100.

Test the NodePort _Service_ using the internal IP of all available nodes and the port 30100 using `curl`, you can reach the internal node IPs directly from your main terminal. On which nodes is the _Service_ reachable? On which node is the _Pod_ running?

##### Answer

First we get an overview:

xxxxxxxxxx

➜ k -n jupiter get all

NAME                                      READY   STATUS    RESTARTS   AGE

pod/jupiter-crew-deploy-8cdf99bc9-klwqt   1/1     Running   0          34m

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE

service/jupiter-crew-svc   ClusterIP   10.100.254.66   <none>        8080/TCP   34m

...

(Optional) Next we check if the ClusterIP _Service_ actually works:

xxxxxxxxxx

➜ k -n jupiter run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 jupiter-crew-svc:8080

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100    45  100    45    0     0   5000      0 --:--:-- --:--:-- --:--:--  5000

<html><body><h1>It works!</h1></body></html>

The _Service_ is working great. Next we change the _Service_ type to NodePort and set the port:

xxxxxxxxxx

k \-n jupiter edit service jupiter-crew-svc

xxxxxxxxxx

\# k -n jupiter edit service jupiter-crew-svc

apiVersion: v1

kind: Service

metadata:

  name: jupiter-crew-svc

  namespace: jupiter

...

spec:

  clusterIP: 10.3.245.70

  ports:

  - name: 8080-80

    port: 8080

    protocol: TCP

    targetPort: 80

    nodePort: 30100 \# add the nodePort

  selector:

    id: jupiter-crew

  sessionAffinity: None

  #type: ClusterIP

  type: NodePort \# change type

status:

  loadBalancer: {}

We check if the _Service_ type was updated:

xxxxxxxxxx

➜ k -n jupiter get svc

NAME               TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE

jupiter-crew-svc   NodePort   10.3.245.70   <none>        8080:30100/TCP   3m52s

(Optional) And we confirm that the service is still reachable internally:

xxxxxxxxxx

➜ k -n jupiter run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 jupiter-crew-svc:8080

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

<html><body><h1>It works!</h1></body></html>

Nice. A NodePort _Service_ kind of lies on top of a ClusterIP one, making the ClusterIP _Service_ reachable on the Node IPs (internal and external). Next we get the _internal_ IPs of all nodes to check the connectivity:

xxxxxxxxxx

➜ k get nodes -o wide

NAME                    STATUS   ROLES          AGE   VERSION   INTERNAL-IP      ...

cluster1-controlplane1  Ready    control-plane  18h   v1.30.1   192.168.100.11   ...

We can test the connection using the node IP:

xxxxxxxxxx

➜ curl 192.168.100.11:30100

<html><body><h1>It works!</h1></body></html>

Here we only have one node in the cluster, but the _Service_ would be reachable on all of them. Even if the _Pod_ is just running on one specific node, the _Service_ makes it available through port 30100 on the internal and external IP addresses of all nodes. This is at least the common/default behaviour but can depend on cluster configuration.

Question 20 | NetworkPolicy
---------------------------

Solve this question on instance: `ssh ckad7326`

In _Namespace_ `venus` you'll find two _Deployments_ named `api` and `frontend`. Both _Deployments_ are exposed inside the cluster using _Services_. Create a _NetworkPolicy_ named `np1` which restricts outgoing tcp connections from _Deployment_ `frontend` and only allows those going to _Deployment_ `api`. Make sure the _NetworkPolicy_ still allows outgoing traffic on UDP/TCP ports 53 for DNS resolution.

Test using: `wget www.google.com` and `wget api:2222` from a _Pod_ of _Deployment_ `frontend`.

##### Answer

> **_INFO:_** For learning NetworkPolicies check out [https://editor.cilium.io](https://editor.cilium.io/). But you're not allowed to use it during the exam.

First we get an overview:

xxxxxxxxxx

➜ k -n venus get all

NAME                            READY   STATUS    RESTARTS   AGE

pod/api-5979b95578-gktxp        1/1     Running   0          57s

pod/api-5979b95578-lhcl5        1/1     Running   0          57s

pod/frontend-789cbdc677-c9v8h   1/1     Running   0          57s

pod/frontend-789cbdc677-npk2m   1/1     Running   0          57s

pod/frontend-789cbdc677-pl67g   1/1     Running   0          57s

pod/frontend-789cbdc677-rjt5r   1/1     Running   0          57s

pod/frontend-789cbdc677-xgf5n   1/1     Running   0          57s

NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE

service/api        ClusterIP   10.3.255.137   <none>        2222/TCP   37s

service/frontend   ClusterIP   10.3.255.135   <none>        80/TCP     57s

...

(Optional) This is not necessary but we could check if the _Services_ are working inside the cluster:

xxxxxxxxxx

➜ k -n venus run tmp --restart=Never --rm -i --image=busybox -i -- wget -O- frontend:80

Connecting to frontend:80 (10.3.245.9:80)

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

...

➜ k -n venus run tmp --restart=Never --rm --image=busybox -i -- wget -O- api:2222

Connecting to api:2222 (10.3.250.233:2222)

<html><body><h1>It works!</h1></body></html>

Then we use any `frontend` _Pod_ and check if it can reach external names and the `api` _Service_:

xxxxxxxxxx

➜ k -n venus exec frontend-789cbdc677-c9v8h -- wget -O- www.google.com

Connecting to www.google.com (216.58.205.227:80)

\-                    100% |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*| 12955  0:00:00 ETA

<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="en"><head>

...

➜ k -n venus exec frontend-789cbdc677-c9v8h -- wget -O- api:2222     

<html><body><h1>It works!</h1></body></html>

Connecting to api:2222 (10.3.255.137:2222)

\-                    100% |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*|    45  0:00:00 ETA

...

We see _Pods_ of `frontend` can reach the `api` and external names.

xxxxxxxxxx

vim 20\_np1.yaml

Now we head to [https://kubernetes.io/docs](https://kubernetes.io/docs), search for _NetworkPolicy_, copy the example code and adjust it to:

xxxxxxxxxx

\# 20\_np1.yaml

apiVersion: networking.k8s.io/v1

kind: NetworkPolicy

metadata:

  name: np1

  namespace: venus

spec:

  podSelector:

    matchLabels:

      id: frontend  \# label of the pods this policy should be applied on

  policyTypes:

  - Egress  \# we only want to control egress

  egress:

  - to:                     \# 1st egress rule

    - podSelector:            \# allow egress only to pods with api label

        matchLabels:

          id: api

  - ports:                  \# 2nd egress rule

    - port: 53  \# allow DNS UDP

      protocol: UDP

    - port: 53  \# allow DNS TCP

      protocol: TCP

Notice that we specify two egress rules in the yaml above. If we specify multiple egress rules then these are connected using a logical OR. So in the example above we do:

xxxxxxxxxx

allow outgoing traffic if

  (destination pod has label id:api) OR ((port is 53 UDP) OR (port is 53 TCP))

Let's have a look at example code which wouldn't work in our case:

xxxxxxxxxx

\# this example does not work in our case

...

  egress:

  - to:                     \# 1st AND ONLY egress rule

    - podSelector:            \# allow egress only to pods with api label

        matchLabels:

          id: api

    ports:                  \# STILL THE SAME RULE but just an additional selector

    - port: 53  \# allow DNS UDP

      protocol: UDP

    - port: 53  \# allow DNS TCP

      protocol: TCP

In the yaml above we only specify one egress rule with two selectors. It can be translated into:

xxxxxxxxxx

allow outgoing traffic if

  (destination pod has label id:api) AND ((port is 53 UDP) OR (port is 53 TCP))

Apply the correct policy:

xxxxxxxxxx

k \-f 20\_np1.yaml create

And try again, external is not working any longer:

xxxxxxxxxx

➜ k -n venus exec frontend-789cbdc677-c9v8h -- wget -O- www.google.de 

Connecting to www.google.de:2222 (216.58.207.67:80)

^C

➜ k -n venus exec frontend-789cbdc677-c9v8h -- wget -O- -T 5 www.google.de:80 

Connecting to www.google.com (172.217.203.104:80)

wget: download timed out

command terminated with exit code 1

Internal connection to `api` work as before:

xxxxxxxxxx

➜ k -n venus exec frontend-789cbdc677-c9v8h -- wget -O- api:2222 

<html><body><h1>It works!</h1></body></html>

Connecting to api:2222 (10.3.255.137:2222)

\-                    100% |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*|    45  0:00:00 ETA

Question 21 | Requests and Limits, ServiceAccount
-------------------------------------------------

Solve this question on instance: `ssh ckad7326`

Team Neptune needs 3 _Pods_ of image `httpd:2.4-alpine`, create a _Deployment_ named `neptune-10ab` for this. The containers should be named `neptune-pod-10ab`. Each container should have a memory request of _20Mi_ and a memory limit of _50Mi_.

Team Neptune has it's own _ServiceAccount_ `neptune-sa-v2` under which the _Pods_ should run. The _Deployment_ should be in _Namespace_ `neptune`.

##### Answer:

xxxxxxxxxx

k \-n neptune create deployment \-h \# help

k \-n neptune create deploy \-h \# deploy is short for deployment

k \-n neptune create deploy neptune-10ab \--image\=httpd:2.4-alpine \--dry-run\=client \-oyaml > 21.yaml

vim 21.yaml

Now make the required changes using vim:

xxxxxxxxxx

\# 21.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  creationTimestamp: null

  labels:

    app: neptune-10ab

  name: neptune-10ab

  namespace: neptune

spec:

  replicas: 3 \# change

  selector:

    matchLabels:

      app: neptune-10ab

  strategy: {}

  template:

    metadata:

      creationTimestamp: null

      labels:

        app: neptune-10ab

    spec:

      serviceAccountName: neptune-sa-v2 \# add

      containers:

      - image: httpd:2.4-alpine

        name: neptune-pod-10ab \# change

        resources:              \# add

          limits:               \# add

            memory: 50Mi  \# add

          requests:             \# add

            memory: 20Mi  \# add

status: {}

Then create the yaml:

xxxxxxxxxx

k create \-f 21.yaml \# namespace already set in yaml

To verify all _Pods_ are running we do:

xxxxxxxxxx

➜ k -n neptune get pod | grep neptune-10ab

neptune-10ab-7d4b8d45b-4nzj5   1/1     Running            0          57s

neptune-10ab-7d4b8d45b-lzwrf   1/1     Running            0          17s

neptune-10ab-7d4b8d45b-z5hcc   1/1     Running            0          17s

Question 22 | Labels, Annotations
---------------------------------

Solve this question on instance: `ssh ckad9043`

Team Sunny needs to identify some of their _Pods_ in namespace `sun`. They ask you to add a new label `protected: true` to all _Pods_ with an existing label `type: worker` or `type: runner`. Also add an annotation `protected: do not delete this pod` to all _Pods_ having the new label `protected: true`.

##### Answer

xxxxxxxxxx

➜ k -n sun get pod --show-labels

NAME           READY   STATUS    RESTARTS   AGE   LABELS

0509649a       1/1     Running   0          25s   type=runner,type\_old=messenger

0509649b       1/1     Running   0          24s   type=worker

1428721e       1/1     Running   0          23s   type=worker

1428721f       1/1     Running   0          22s   type=worker

43b9a          1/1     Running   0          22s   type=test

4c09           1/1     Running   0          21s   type=worker

4c35           1/1     Running   0          20s   type=worker

4fe4           1/1     Running   0          19s   type=worker

5555a          1/1     Running   0          19s   type=messenger

86cda          1/1     Running   0          18s   type=runner

8d1c           1/1     Running   0          17s   type=messenger

a004a          1/1     Running   0          16s   type=runner

a94128196      1/1     Running   0          15s   type=runner,type\_old=messenger

afd79200c56a   1/1     Running   0          15s   type=worker

b667           1/1     Running   0          14s   type=worker

fdb2           1/1     Running   0          13s   type=worker

If we would only like to get pods with certain labels we can run:

xxxxxxxxxx

k \-n sun get pod \-l type\=runner \# only pods with label runner

We can use this label filtering also when using other commands, like setting new labels:

xxxxxxxxxx

k label \-h \# help

k \-n sun label pod \-l type\=runner protected\=true \# run for label runner

k \-n sun label pod \-l type\=worker protected\=true \# run for label worker

Or we could run:

xxxxxxxxxx

k \-n sun label pod \-l "type in (worker,runner)" protected\=true

Let's check the result:

xxxxxxxxxx

➜ k -n sun get pod --show-labels

NAME           ...   AGE   LABELS

0509649a       ...          56s   protected=true,type=runner,type\_old=messenger

0509649b       ...          55s   protected=true,type=worker

1428721e       ...          54s   protected=true,type=worker

1428721f       ...          53s   protected=true,type=worker

43b9a          ...          53s   type=test

4c09           ...          52s   protected=true,type=worker

4c35           ...          51s   protected=true,type=worker

4fe4           ...          50s   protected=true,type=worker

5555a          ...          50s   type=messenger

86cda          ...          49s   protected=true,type=runner

8d1c           ...          48s   type=messenger

a004a          ...          47s   protected=true,type=runner

a94128196      ...          46s   protected=true,type=runner,type\_old=messenger

afd79200c56a   ...          46s   protected=true,type=worker

b667           ...          45s   protected=true,type=worker

fdb2           ...          44s   protected=true,type=worker

Looking good. Finally we set the annotation using the newly assigned label `protected: true`:

xxxxxxxxxx

k \-n sun annotate pod \-l protected\=true protected\="do not delete this pod"

Not requested in the task but for your own control you could run:

xxxxxxxxxx

k \-n sun get pod \-l protected\=true \-o yaml | grep \-A 8 metadata: 

 html {overflow-x: initial !important;}:root { --bg-color: #ffffff; --text-color: #333333; --select-text-bg-color: #B5D6FC; --select-text-font-color: auto; --monospace: "Lucida Console",Consolas,"Courier",monospace; --title-bar-height: 20px; } .mac-os-11 { --title-bar-height: 28px; } html { font-size: 14px; background-color: var(--bg-color); color: var(--text-color); font-family: "Helvetica Neue", Helvetica, Arial, sans-serif; -webkit-font-smoothing: antialiased; } h1, h2, h3, h4, h5 { white-space: pre-wrap; } body { margin: 0px; padding: 0px; height: auto; inset: 0px; font-size: 1rem; line-height: 1.42857143; overflow-x: hidden; background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; background-position: inherit; background-repeat: inherit; } iframe { margin: auto; } a.url { word-break: break-all; } a:active, a:hover { outline: 0px; } .in-text-selection, ::selection { text-shadow: none; background: var(--select-text-bg-color); color: var(--select-text-font-color); } #write { margin: 0px auto; height: auto; width: inherit; word-break: normal; word-wrap: break-word; position: relative; white-space: normal; overflow-x: visible; padding-top: 36px; } #write.first-line-indent p { text-indent: 2em; } #write.first-line-indent li p, #write.first-line-indent p \* { text-indent: 0px; } #write.first-line-indent li { margin-left: 2em; } .for-image #write { padding-left: 8px; padding-right: 8px; } body.typora-export { padding-left: 30px; padding-right: 30px; } .typora-export .footnote-line, .typora-export li, .typora-export p { white-space: pre-wrap; } .typora-export .task-list-item input { pointer-events: none; } @media screen and (max-width: 500px) { body.typora-export { padding-left: 0px; padding-right: 0px; } #write { padding-left: 20px; padding-right: 20px; } } #write li > figure:last-child { margin-bottom: 0.5rem; } #write ol, #write ul { position: relative; } img { max-width: 100%; vertical-align: middle; image-orientation: from-image; } button, input, select, textarea { color: inherit; font-family: inherit; font-size: inherit; font-style: inherit; font-variant-caps: inherit; font-weight: inherit; font-stretch: inherit; line-height: inherit; } input\[type="checkbox"\], input\[type="radio"\] { line-height: normal; padding: 0px; } \*, ::after, ::before { box-sizing: border-box; } #write h1, #write h2, #write h3, #write h4, #write h5, #write h6, #write p, #write pre { width: inherit; } #write h1, #write h2, #write h3, #write h4, #write h5, #write h6, #write p { position: relative; } p { line-height: inherit; } h1, h2, h3, h4, h5, h6 { break-after: avoid-page; break-inside: avoid; orphans: 4; } p { orphans: 4; } h1 { font-size: 2rem; } h2 { font-size: 1.8rem; } h3 { font-size: 1.6rem; } h4 { font-size: 1.4rem; } h5 { font-size: 1.2rem; } h6 { font-size: 1rem; } .md-math-block, .md-rawblock, h1, h2, h3, h4, h5, h6, p { margin-top: 1rem; margin-bottom: 1rem; } .hidden { display: none; } .md-blockmeta { color: rgb(204, 204, 204); font-weight: 700; font-style: italic; } a { cursor: pointer; } sup.md-footnote { padding: 2px 4px; background-color: rgba(238, 238, 238, 0.7); color: rgb(85, 85, 85); border-top-left-radius: 4px; border-top-right-radius: 4px; border-bottom-right-radius: 4px; border-bottom-left-radius: 4px; cursor: pointer; } sup.md-footnote a, sup.md-footnote a:hover { color: inherit; text-transform: inherit; text-decoration: inherit; } #write input\[type="checkbox"\] { cursor: pointer; width: inherit; height: inherit; } figure { overflow-x: auto; margin: 1.2em 0px; max-width: calc(100% + 16px); padding: 0px; } figure > table { margin: 0px; } thead, tr { break-inside: avoid; break-after: auto; } thead { display: table-header-group; } table { border-collapse: collapse; border-spacing: 0px; width: 100%; overflow: auto; break-inside: auto; text-align: left; } table.md-table td { min-width: 32px; } .CodeMirror-gutters { border-right-width: 0px; background-color: inherit; } .CodeMirror-linenumber { -webkit-user-select: none; } .CodeMirror { text-align: left; } .CodeMirror-placeholder { opacity: 0.3; } .CodeMirror pre { padding: 0px 4px; } .CodeMirror-lines { padding: 0px; } div.hr:focus { cursor: none; } #write pre { white-space: pre-wrap; } #write.fences-no-line-wrapping pre { white-space: pre; } #write pre.ty-contain-cm { white-space: normal; } .CodeMirror-gutters { margin-right: 4px; } .md-fences { font-size: 0.9rem; display: block; break-inside: avoid; text-align: left; overflow: visible; white-space: pre; background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; position: relative !important; background-position: inherit; background-repeat: inherit; } .md-fences-adv-panel { width: 100%; margin-top: 10px; text-align: center; padding-top: 0px; padding-bottom: 8px; overflow-x: auto; } #write .md-fences.mock-cm { white-space: pre-wrap; } .md-fences.md-fences-with-lineno { padding-left: 0px; } #write.fences-no-line-wrapping .md-fences.mock-cm { white-space: pre; overflow-x: auto; } .md-fences.mock-cm.md-fences-with-lineno { padding-left: 8px; } .CodeMirror-line, twitterwidget { break-inside: avoid; } svg { break-inside: avoid; } .footnotes { opacity: 0.8; font-size: 0.9rem; margin-top: 1em; margin-bottom: 1em; } .footnotes + .footnotes { margin-top: 0px; } .md-reset { margin: 0px; padding: 0px; border: 0px; outline: 0px; vertical-align: top; text-decoration: none; text-shadow: none; float: none; position: static; width: auto; height: auto; white-space: nowrap; cursor: inherit; line-height: normal; font-weight: 400; text-align: left; box-sizing: content-box; direction: ltr; background-position: 0px 0px; } li div { padding-top: 0px; } blockquote { margin: 1rem 0px; } li .mathjax-block, li p { margin: 0.5rem 0px; } li blockquote { margin: 1rem 0px; } li { margin: 0px; position: relative; } blockquote > :last-child { margin-bottom: 0px; } blockquote > :first-child, li > :first-child { margin-top: 0px; } .footnotes-area { color: rgb(136, 136, 136); margin-top: 0.714rem; padding-bottom: 0.143rem; white-space: normal; } #write .footnote-line { white-space: pre-wrap; } @media print { body, html { border: 1px solid transparent; height: 99%; break-after: avoid; break-before: avoid; font-variant-ligatures: no-common-ligatures; } #write { margin-top: 0px; border-color: transparent !important; padding-top: 0px !important; padding-bottom: 0px !important; } .typora-export \* { -webkit-print-color-adjust: exact; } .typora-export #write { break-after: avoid; } .typora-export #write::after { height: 0px; } .is-mac table { break-inside: avoid; } #write > p:nth-child(1) { margin-top: 0px; } .typora-export-show-outline .typora-export-sidebar { display: none; } figure { overflow-x: visible; } } .footnote-line { margin-top: 0.714em; font-size: 0.7em; } a img, img a { cursor: pointer; } pre.md-meta-block { font-size: 0.8rem; min-height: 0.8rem; white-space: pre-wrap; background-color: rgb(204, 204, 204); display: block; overflow-x: hidden; } p > .md-image:only-child:not(.md-img-error) img, p > img:only-child { display: block; margin: auto; } #write.first-line-indent p > .md-image:only-child:not(.md-img-error) img { left: -2em; position: relative; } p > .md-image:only-child { display: inline-block; width: 100%; } #write .MathJax\_Display { margin: 0.8em 0px 0px; } .md-math-block { width: 100%; } .md-math-block:not(:empty)::after { display: none; } .MathJax\_ref { fill: currentcolor; } \[contenteditable="true"\]:active, \[contenteditable="true"\]:focus, \[contenteditable="false"\]:active, \[contenteditable="false"\]:focus { outline: 0px; box-shadow: none; } .md-task-list-item { position: relative; list-style-type: none; } .task-list-item.md-task-list-item { padding-left: 0px; } .md-task-list-item > input { position: absolute; top: 0px; left: 0px; margin-left: -1.2em; margin-top: calc(1em - 10px); border: none; } .math { font-size: 1rem; } .md-toc { min-height: 3.58rem; position: relative; font-size: 0.9rem; border-top-left-radius: 10px; border-top-right-radius: 10px; border-bottom-right-radius: 10px; border-bottom-left-radius: 10px; } .md-toc-content { position: relative; margin-left: 0px; } .md-toc-content::after, .md-toc::after { display: none; } .md-toc-item { display: block; color: rgb(65, 131, 196); } .md-toc-item a { text-decoration: none; } .md-toc-inner:hover { text-decoration: underline; } .md-toc-inner { display: inline-block; cursor: pointer; } .md-toc-h1 .md-toc-inner { margin-left: 0px; font-weight: 700; } .md-toc-h2 .md-toc-inner { margin-left: 2em; } .md-toc-h3 .md-toc-inner { margin-left: 4em; } .md-toc-h4 .md-toc-inner { margin-left: 6em; } .md-toc-h5 .md-toc-inner { margin-left: 8em; } .md-toc-h6 .md-toc-inner { margin-left: 10em; } @media screen and (max-width: 48em) { .md-toc-h3 .md-toc-inner { margin-left: 3.5em; } .md-toc-h4 .md-toc-inner { margin-left: 5em; } .md-toc-h5 .md-toc-inner { margin-left: 6.5em; } .md-toc-h6 .md-toc-inner { margin-left: 8em; } } a.md-toc-inner { font-size: inherit; font-style: inherit; font-weight: inherit; line-height: inherit; } .footnote-line a:not(.reversefootnote) { color: inherit; } .reversefootnote { font-family: ui-monospace, sans-serif; } .md-attr { display: none; } .md-fn-count::after { content: "."; } code, pre, samp, tt { font-family: var(--monospace); } kbd { margin: 0px 0.1em; padding: 0.1em 0.6em; font-size: 0.8em; color: rgb(36, 39, 41); background-color: rgb(255, 255, 255); border: 1px solid rgb(173, 179, 185); border-top-left-radius: 3px; border-top-right-radius: 3px; border-bottom-right-radius: 3px; border-bottom-left-radius: 3px; box-shadow: rgba(12, 13, 14, 0.2) 0px 1px 0px, rgb(255, 255, 255) 0px 0px 0px 2px inset; white-space: nowrap; vertical-align: middle; } .md-comment { color: rgb(162, 127, 3); opacity: 0.6; font-family: var(--monospace); } code { text-align: left; } a.md-print-anchor { white-space: pre !important; border: none !important; display: inline-block !important; position: absolute !important; width: 1px !important; right: 0px !important; outline: 0px !important; text-shadow: initial !important; background-position: 0px 0px !important; } .os-windows.monocolor-emoji .md-emoji { font-family: "Segoe UI Symbol", sans-serif; } .md-diagram-panel > svg { max-width: 100%; } \[lang="flow"\] svg, \[lang="mermaid"\] svg { max-width: 100%; height: auto; } \[lang="mermaid"\] .node text { font-size: 1rem; } table tr th { border-bottom-width: 0px; } video { max-width: 100%; display: block; margin: 0px auto; } iframe { max-width: 100%; width: 100%; border: none; } .highlight td, .highlight tr { border: 0px; } mark { background-color: rgb(255, 255, 0); color: rgb(0, 0, 0); } .md-html-inline .md-plain, .md-html-inline strong, mark .md-inline-math, mark strong { color: inherit; } .md-expand mark .md-meta { opacity: 0.3 !important; } mark .md-meta { color: rgb(0, 0, 0); } @media print { .typora-export h1, .typora-export h2, .typora-export h3, .typora-export h4, .typora-export h5, .typora-export h6 { break-inside: avoid; } } .md-diagram-panel .messageText { stroke: none !important; } .md-diagram-panel .start-state { fill: var(--node-fill); } .md-diagram-panel .edgeLabel rect { opacity: 1 !important; } .md-fences.md-fences-math { font-size: 1em; } .md-fences-advanced:not(.md-focus) { padding: 0px; white-space: nowrap; border: 0px; } .md-fences-advanced:not(.md-focus) { background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; background-position: inherit; background-repeat: inherit; } .typora-export-show-outline .typora-export-content { max-width: 1440px; margin: auto; display: flex; flex-direction: row; } .typora-export-sidebar { width: 300px; font-size: 0.8rem; margin-top: 80px; margin-right: 18px; } .typora-export-show-outline #write { --webkit-flex: 2; flex: 2 1 0%; } .typora-export-sidebar .outline-content { position: fixed; top: 0px; max-height: 100%; overflow: hidden auto; padding-bottom: 30px; padding-top: 60px; width: 300px; } @media screen and (max-width: 1024px) { .typora-export-sidebar, .typora-export-sidebar .outline-content { width: 240px; } } @media screen and (max-width: 800px) { .typora-export-sidebar { display: none; } } .outline-content li, .outline-content ul { margin-left: 0px; margin-right: 0px; padding-left: 0px; padding-right: 0px; list-style: none; } .outline-content ul { margin-top: 0px; margin-bottom: 0px; } .outline-content strong { font-weight: 400; } .outline-expander { width: 1rem; height: 1.428571429rem; position: relative; display: table-cell; vertical-align: middle; cursor: pointer; padding-left: 4px; } .outline-expander::before { content: ""; position: relative; font-family: Ionicons; display: inline-block; font-size: 8px; vertical-align: middle; } .outline-item { padding-top: 3px; padding-bottom: 3px; cursor: pointer; } .outline-expander:hover::before { content: ""; } .outline-h1 > .outline-item { padding-left: 0px; } .outline-h2 > .outline-item { padding-left: 1em; } .outline-h3 > .outline-item { padding-left: 2em; } .outline-h4 > .outline-item { padding-left: 3em; } .outline-h5 > .outline-item { padding-left: 4em; } .outline-h6 > .outline-item { padding-left: 5em; } .outline-label { cursor: pointer; display: table-cell; vertical-align: middle; text-decoration: none; color: inherit; } .outline-label:hover { text-decoration: underline; } .outline-item:hover { border-color: rgb(245, 245, 245); background-color: var(--item-hover-bg-color); } .outline-item:hover { margin-left: -28px; margin-right: -28px; border-left-width: 28px; border-left-style: solid; border-left-color: transparent; border-right-width: 28px; border-right-style: solid; border-right-color: transparent; } .outline-item-single .outline-expander::before, .outline-item-single .outline-expander:hover::before { display: none; } .outline-item-open > .outline-item > .outline-expander::before { content: ""; } .outline-children { display: none; } .info-panel-tab-wrapper { display: none; } .outline-item-open > .outline-children { display: block; } .typora-export .outline-item { padding-top: 1px; padding-bottom: 1px; } .typora-export .outline-item:hover { margin-right: -8px; border-right-width: 8px; border-right-style: solid; border-right-color: transparent; } .typora-export .outline-expander::before { content: "+"; font-family: inherit; top: -1px; } .typora-export .outline-expander:hover::before, .typora-export .outline-item-open > .outline-item > .outline-expander::before { content: "−"; } .typora-export-collapse-outline .outline-children { display: none; } .typora-export-collapse-outline .outline-item-open > .outline-children, .typora-export-no-collapse-outline .outline-children { display: block; } .typora-export-no-collapse-outline .outline-expander::before { content: "" !important; } .typora-export-show-outline .outline-item-active > .outline-item .outline-label { font-weight: 700; } .md-inline-math-container mjx-container { zoom: 0.95; } mjx-container { break-inside: avoid; } .md-alert.md-alert-note { border-left-color: rgb(9, 105, 218); } .md-alert.md-alert-important { border-left-color: rgb(130, 80, 223); } .md-alert.md-alert-warning { border-left-color: rgb(154, 103, 0); } .md-alert.md-alert-tip { border-left-color: rgb(31, 136, 61); } .md-alert.md-alert-caution { border-left-color: rgb(207, 34, 46); } .md-alert { padding: 0px 1em; margin-bottom: 16px; color: inherit; border-left-width: 0.25em; border-left-style: solid; border-left-color: rgb(0, 0, 0); } .md-alert-text-note { color: rgb(9, 105, 218); } .md-alert-text-important { color: rgb(130, 80, 223); } .md-alert-text-warning { color: rgb(154, 103, 0); } .md-alert-text-tip { color: rgb(31, 136, 61); } .md-alert-text-caution { color: rgb(207, 34, 46); } .md-alert-text { font-size: 0.9rem; font-weight: 700; } .md-alert-text svg { fill: currentcolor; position: relative; top: 0.125em; margin-right: 1ch; overflow: visible; } .md-alert-text-container::after { content: attr(data-text); text-transform: capitalize; pointer-events: none; margin-right: 1ch; } .CodeMirror { height: auto; } .CodeMirror.cm-s-inner { background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; background-position: inherit; background-repeat: inherit; } .CodeMirror-scroll { overflow: auto hidden; z-index: 3; } .CodeMirror-gutter-filler, .CodeMirror-scrollbar-filler { background-color: rgb(255, 255, 255); } .CodeMirror-gutters { border-right-width: 1px; border-right-style: solid; border-right-color: rgb(221, 221, 221); background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; white-space: nowrap; background-position: inherit; background-repeat: inherit; } .CodeMirror-linenumber { padding: 0px 3px 0px 5px; text-align: right; color: rgb(153, 153, 153); } .cm-s-inner .cm-keyword { color: rgb(119, 0, 136); } .cm-s-inner .cm-atom, .cm-s-inner.cm-atom { color: rgb(34, 17, 153); } .cm-s-inner .cm-number { color: rgb(17, 102, 68); } .cm-s-inner .cm-def { color: rgb(0, 0, 255); } .cm-s-inner .cm-variable { color: rgb(0, 0, 0); } .cm-s-inner .cm-variable-2 { color: rgb(0, 85, 170); } .cm-s-inner .cm-variable-3 { color: rgb(0, 136, 85); } .cm-s-inner .cm-string { color: rgb(170, 17, 17); } .cm-s-inner .cm-property { color: rgb(0, 0, 0); } .cm-s-inner .cm-operator { color: rgb(152, 26, 26); } .cm-s-inner .cm-comment, .cm-s-inner.cm-comment { color: rgb(170, 85, 0); } .cm-s-inner .cm-string-2 { color: rgb(255, 85, 0); } .cm-s-inner .cm-meta { color: rgb(85, 85, 85); } .cm-s-inner .cm-qualifier { color: rgb(85, 85, 85); } .cm-s-inner .cm-builtin { color: rgb(51, 0, 170); } .cm-s-inner .cm-bracket { color: rgb(153, 153, 119); } .cm-s-inner .cm-tag { color: rgb(17, 119, 0); } .cm-s-inner .cm-attribute { color: rgb(0, 0, 204); } .cm-s-inner .cm-header, .cm-s-inner.cm-header { color: rgb(0, 0, 255); } .cm-s-inner .cm-quote, .cm-s-inner.cm-quote { color: rgb(0, 153, 0); } .cm-s-inner .cm-hr, .cm-s-inner.cm-hr { color: rgb(153, 153, 153); } .cm-s-inner .cm-link, .cm-s-inner.cm-link { color: rgb(0, 0, 204); } .cm-negative { color: rgb(221, 68, 68); } .cm-positive { color: rgb(34, 153, 34); } .cm-header, .cm-strong { font-weight: 700; } .cm-del { text-decoration: line-through; } .cm-em { font-style: italic; } .cm-link { text-decoration: underline; } .cm-error { color: red; } .cm-invalidchar { color: red; } .cm-constant { color: rgb(38, 139, 210); } .cm-defined { color: rgb(181, 137, 0); } div.CodeMirror span.CodeMirror-matchingbracket { color: rgb(0, 255, 0); } div.CodeMirror span.CodeMirror-nonmatchingbracket { color: rgb(255, 34, 34); } .cm-s-inner .CodeMirror-activeline-background { background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; background-position: inherit; background-repeat: inherit; } .CodeMirror { position: relative; overflow: hidden; } .CodeMirror-scroll { height: 100%; outline: 0px; position: relative; box-sizing: content-box; background-image: inherit; background-size: inherit; background-attachment: inherit; background-origin: inherit; background-clip: inherit; background-color: inherit; background-position: inherit; background-repeat: inherit; } .CodeMirror-sizer { position: relative; } .CodeMirror-gutter-filler, .CodeMirror-hscrollbar, .CodeMirror-scrollbar-filler, .CodeMirror-vscrollbar { position: absolute; z-index: 6; display: none; outline: 0px; } .CodeMirror-vscrollbar { right: 0px; top: 0px; overflow: hidden; } .CodeMirror-hscrollbar { bottom: 0px; left: 0px; overflow: auto hidden; } .CodeMirror-scrollbar-filler { right: 0px; bottom: 0px; } .CodeMirror-gutter-filler { left: 0px; bottom: 0px; } .CodeMirror-gutters { position: absolute; left: 0px; top: 0px; padding-bottom: 10px; z-index: 3; overflow-y: hidden; } .CodeMirror-gutter { white-space: normal; height: 100%; box-sizing: content-box; padding-bottom: 30px; margin-bottom: -32px; display: inline-block; } .CodeMirror-gutter-wrapper { position: absolute; z-index: 4; border: none !important; background-position: 0px 0px !important; } .CodeMirror-gutter-background { position: absolute; top: 0px; bottom: 0px; z-index: 4; } .CodeMirror-gutter-elt { position: absolute; cursor: default; z-index: 4; } .CodeMirror-lines { cursor: text; } .CodeMirror pre { border-top-left-radius: 0px; border-top-right-radius: 0px; border-bottom-right-radius: 0px; border-bottom-left-radius: 0px; border-width: 0px; font-family: inherit; font-size: inherit; margin: 0px; white-space: pre; word-wrap: normal; color: inherit; z-index: 2; position: relative; overflow: visible; background-position: 0px 0px; } .CodeMirror-wrap pre { word-wrap: break-word; white-space: pre-wrap; word-break: normal; } .CodeMirror-code pre { border-right-width: 30px; border-right-style: solid; border-right-color: transparent; width: fit-content; } .CodeMirror-wrap .CodeMirror-code pre { border-right-style: none; width: auto; } .CodeMirror-linebackground { position: absolute; inset: 0px; z-index: 0; } .CodeMirror-linewidget { position: relative; z-index: 2; overflow: auto; } .CodeMirror-wrap .CodeMirror-scroll { overflow-x: hidden; } .CodeMirror-measure { position: absolute; width: 100%; height: 0px; overflow: hidden; visibility: hidden; } .CodeMirror-measure pre { position: static; } .CodeMirror div.CodeMirror-cursor { position: absolute; visibility: hidden; border-right-style: none; width: 0px; } .CodeMirror div.CodeMirror-cursor { visibility: hidden; } .CodeMirror-focused div.CodeMirror-cursor { visibility: inherit; } .cm-searching { background-color: rgba(255, 255, 0, 0.4); } span.cm-underlined { text-decoration: underline; } span.cm-strikethrough { text-decoration: line-through; } .cm-tw-syntaxerror { color: rgb(255, 255, 255); background-color: rgb(153, 0, 0); } .cm-tw-deleted { text-decoration: line-through; } .cm-tw-header5 { font-weight: 700; } .cm-tw-listitem:first-child { padding-left: 10px; } .cm-tw-box { border-style: solid; border-right-width: 1px; border-bottom-width: 1px; border-left-width: 1px; border-color: inherit; border-top-width: 0px !important; } .cm-tw-underline { text-decoration: underline; } @media print { .CodeMirror div.CodeMirror-cursor { visibility: hidden; } } :root { --side-bar-bg-color: #fafafa; --control-text-color: #777; } @include-when-export url(https://fonts.googleapis.com/css?family=Open+Sans:400italic,700italic,700,400&subset=latin,latin-ext); /\* open-sans-regular - latin-ext\_latin \*/ /\* open-sans-italic - latin-ext\_latin \*/ /\* open-sans-700 - latin-ext\_latin \*/ /\* open-sans-700italic - latin-ext\_latin \*/ html { font-size: 16px; -webkit-font-smoothing: antialiased; } body { font-family: "Open Sans","Clear Sans", "Helvetica Neue", Helvetica, Arial, 'Segoe UI Emoji', sans-serif; color: rgb(51, 51, 51); line-height: 1.6; } #write { max-width: 860px; margin: 0 auto; padding: 30px; padding-bottom: 100px; } @media only screen and (min-width: 1400px) { #write { max-width: 1024px; } } @media only screen and (min-width: 1800px) { #write { max-width: 1200px; } } #write > ul:first-child, #write > ol:first-child{ margin-top: 30px; } a { color: #4183C4; } h1, h2, h3, h4, h5, h6 { position: relative; margin-top: 1rem; margin-bottom: 1rem; font-weight: bold; line-height: 1.4; cursor: text; } h1:hover a.anchor, h2:hover a.anchor, h3:hover a.anchor, h4:hover a.anchor, h5:hover a.anchor, h6:hover a.anchor { text-decoration: none; } h1 tt, h1 code { font-size: inherit; } h2 tt, h2 code { font-size: inherit; } h3 tt, h3 code { font-size: inherit; } h4 tt, h4 code { font-size: inherit; } h5 tt, h5 code { font-size: inherit; } h6 tt, h6 code { font-size: inherit; } h1 { font-size: 2.25em; line-height: 1.2; border-bottom: 1px solid #eee; } h2 { font-size: 1.75em; line-height: 1.225; border-bottom: 1px solid #eee; } /\*@media print { .typora-export h1, .typora-export h2 { border-bottom: none; padding-bottom: initial; } .typora-export h1::after, .typora-export h2::after { content: ""; display: block; height: 100px; margin-top: -96px; border-top: 1px solid #eee; } }\*/ h3 { font-size: 1.5em; line-height: 1.43; } h4 { font-size: 1.25em; } h5 { font-size: 1em; } h6 { font-size: 1em; color: #777; } p, blockquote, ul, ol, dl, table{ margin: 0.8em 0; } li>ol, li>ul { margin: 0 0; } hr { height: 2px; padding: 0; margin: 16px 0; background-color: #e7e7e7; border: 0 none; overflow: hidden; box-sizing: content-box; } li p.first { display: inline-block; } ul, ol { padding-left: 30px; } ul:first-child, ol:first-child { margin-top: 0; } ul:last-child, ol:last-child { margin-bottom: 0; } blockquote { border-left: 4px solid #dfe2e5; padding: 0 15px; color: #777777; } blockquote blockquote { padding-right: 0; } table { padding: 0; word-break: initial; } table tr { border: 1px solid #dfe2e5; margin: 0; padding: 0; } table tr:nth-child(2n), thead { background-color: #f8f8f8; } table th { font-weight: bold; border: 1px solid #dfe2e5; border-bottom: 0; margin: 0; padding: 6px 13px; } table td { border: 1px solid #dfe2e5; margin: 0; padding: 6px 13px; } table th:first-child, table td:first-child { margin-top: 0; } table th:last-child, table td:last-child { margin-bottom: 0; } .CodeMirror-lines { padding-left: 4px; } .code-tooltip { box-shadow: 0 1px 1px 0 rgba(0,28,36,.3); border-top: 1px solid #eef2f2; } .md-fences, code, tt { border: 1px solid #e7eaed; background-color: #f8f8f8; border-radius: 3px; padding: 0; padding: 2px 4px 0px 4px; font-size: 0.9em; } code { background-color: #f3f4f4; padding: 0 2px 0 2px; } .md-fences { margin-bottom: 15px; margin-top: 15px; padding-top: 8px; padding-bottom: 6px; } .md-task-list-item > input { margin-left: -1.3em; } @media print { html { font-size: 13px; } pre { page-break-inside: avoid; word-wrap: break-word; } } .md-fences { background-color: #f8f8f8; } #write pre.md-meta-block { padding: 1rem; font-size: 85%; line-height: 1.45; background-color: #f7f7f7; border: 0; border-radius: 3px; color: #777777; margin-top: 0 !important; } .mathjax-block>.code-tooltip { bottom: .375rem; } .md-mathjax-midline { background: #fafafa; } #write>h3.md-focus:before{ left: -1.5625rem; top: .375rem; } #write>h4.md-focus:before{ left: -1.5625rem; top: .285714286rem; } #write>h5.md-focus:before{ left: -1.5625rem; top: .285714286rem; } #write>h6.md-focus:before{ left: -1.5625rem; top: .285714286rem; } .md-image>.md-meta { /\*border: 1px solid #ddd;\*/ border-radius: 3px; padding: 2px 0px 0px 4px; font-size: 0.9em; color: inherit; } .md-tag { color: #a7a7a7; opacity: 1; } .md-toc { margin-top:20px; padding-bottom:20px; } .sidebar-tabs { border-bottom: none; } #typora-quick-open { border: 1px solid #ddd; background-color: #f8f8f8; } #typora-quick-open-item { background-color: #FAFAFA; border-color: #FEFEFE #e5e5e5 #e5e5e5 #eee; border-style: solid; border-width: 1px; } /\*\* focus mode \*/ .on-focus-mode blockquote { border-left-color: rgba(85, 85, 85, 0.12); } header, .context-menu, .megamenu-content, footer{ font-family: "Segoe UI", "Arial", sans-serif; } .file-node-content:hover .file-node-icon, .file-node-content:hover .file-node-open-state{ visibility: visible; } .mac-seamless-mode #typora-sidebar { background-color: #fafafa; background-color: var(--side-bar-bg-color); } .mac-os #write{ caret-color: AccentColor; } .md-lang { color: #b4654d; } /\*.html-for-mac { --item-hover-bg-color: #E6F0FE; }\*/ #md-notification .btn { border: 0; } .dropdown-menu .divider { border-color: #e5e5e5; opacity: 0.4; } .ty-preferences .window-content { background-color: #fafafa; } .ty-preferences .nav-group-item.active { color: white; background: #999; } .menu-item-container a.menu-style-btn { background-color: #f5f8fa; background-image: linear-gradient( 180deg , hsla(0, 0%, 100%, 0.8), hsla(0, 0%, 100%, 0)); } pre\[lang="bash"\] { background-color: #36454b; } pre\[lang="bash"\] .cm-s-inner.CodeMirror { background-color: #36454b; color: #EEFFFF; } pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-attribute, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-operator, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-string, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-atom, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-number, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-keyword, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-def, pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-builtin { color: #EEFFFF; } pre\[lang="bash"\] .cm-s-inner.CodeMirror .cm-comment { color: #F78C6C; } pre\[lang="term"\] { background-color: #1c252b; } pre\[lang="term"\] .cm-s-inner.CodeMirror { background-color: #1c252b; color: #EEFFFF; } pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-attribute, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-operator, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-string, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-atom, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-number, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-keyword, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-def, pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-builtin { color: #EEFFFF; } pre\[lang="term"\] .cm-s-inner.CodeMirror .cm-comment { color: #F78C6C; } /\* neo theme for codemirror \*/ /\* Color scheme \*/ pre\[lang="yaml"\] .cm-s-inner.CodeMirror { background-color:#ffffff; color:#2e383c; line-height:1.4375; } pre\[lang="yaml"\] .cm-s-inner .cm-comment { color:#F78C6C; } pre\[lang="yaml"\] .cm-s-inner .cm-keyword, pre\[lang="yaml"\] .cm-s-inner .cm-property { color:#1d75b3; } pre\[lang="yaml"\] .cm-s-inner .cm-atom,pre\[lang="yaml"\] .cm-s-inner .cm-number { color:#75438a; } pre\[lang="yaml"\] .cm-s-inner .cm-node,pre\[lang="yaml"\] .cm-s-inner .cm-tag { color:#9c3328; } pre\[lang="yaml"\] .cm-s-inner .cm-string { color:#b35e14; } pre\[lang="yaml"\] .cm-s-inner .cm-variable,pre\[lang="yaml"\] .cm-s-inner .cm-qualifier { color:#047d65; } /\* Editor styling \*/ pre\[lang="yaml"\] .cm-s-inner pre { padding:0; } pre\[lang="yaml"\] .cm-s-inner .CodeMirror-gutters { border:none; border-right:10px solid transparent; background-color:transparent; } pre\[lang="yaml"\] .cm-s-inner .CodeMirror-linenumber { padding:0; color:#e0e2e5; } pre\[lang="yaml"\] .cm-s-inner .CodeMirror-guttermarker { color: #1d75b3; } pre\[lang="yaml"\] .cm-s-inner .CodeMirror-guttermarker-subtle { color: #e0e2e5; } pre\[lang="yaml"\] .cm-s-inner .CodeMirror-cursor { width: auto; border: 0; background: rgba(155,157,162,0.37); z-index: 1; } @media print { @page {margin: 0 0 0 0;} body.typora-export {padding-left: 0; padding-right: 0;} #write {padding:0;}} preview

CKAD Simulator Preview Kubernetes 1.30
======================================

[https://killer.sh](https://killer.sh/)

This is a preview of the full CKAD Simulator course content.

The full course contains 22 questions and scenarios which cover all the CKAD areas. The course also provides a browser terminal which is a very close replica of the original one. This is great to get used and comfortable before the real exam. After the test session (120 minutes), or if you stop it early, you'll get access to all questions and their detailed solutions. You'll have 36 hours cluster access in total which means even after the session, once you have the solutions, you can still play around.

The following preview will give you an idea of what the full course will provide. These preview questions are not part of the 22 in the full course but in addition to it. But the preview questions are part of the same CKAD simulation environment which we setup for you, so with access to the full course you can solve these too.

The answers provided here assume that you did run the initial terminal setup suggestions as provided in the tips section, but especially:

**These questions can be solved in the test environment provided through the CKAD Simulator**

Preview Question 1
------------------

Solve this question on instance: `ssh ckad9043`

In _Namespace_ `pluto` there is a _Deployment_ named `project-23-api`. It has been working okay for a while but Team Pluto needs it to be more reliable. Implement a liveness-probe which checks the container to be reachable on port 80. Initially the probe should wait _10_, periodically _15_ seconds.

The original _Deployment_ yaml is available at `/opt/course/p1/project-23-api.yaml`. Save your changes at `/opt/course/p1/project-23-api-new.yaml` and apply the changes.

##### Answer

First we get an overview:

​x

➜ k -n pluto get all -o wide

NAME                                  READY   STATUS    ... IP           ...

pod/holy-api                          1/1     Running   ... 10.12.0.26   ...

pod/project-23-api-784857f54c-dx6h6   1/1     Running   ... 10.12.2.15   ...

pod/project-23-api-784857f54c-sj8df   1/1     Running   ... 10.12.1.18   ...

pod/project-23-api-784857f54c-t4xmh   1/1     Running   ... 10.12.0.23   ...

NAME                             READY   UP-TO-DATE   AVAILABLE   ...

deployment.apps/project-23-api   3/3     3            3           ...

To note: we see another _Pod_ here called `holy-api` which is part of another section. This is often the case in the provided scenarios, so be careful to only manipulate the resources you need to. Just like in the real world and in the exam.

Next we use `nginx:alpine` and `curl` to check if one _Pod_ is accessible on port 80:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 10.12.2.15

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

...

We could also use `busybox` and `wget` for this:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm --image=busybox -i -- wget -O- 10.12.2.15

Connecting to 10.12.2.15 (10.12.2.15:80)

writing to stdout

\-                    100% |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*|   612  0:00:00 ETA

written to stdout

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

Now that we're sure the _Deployment_ works we can continue with altering the provided yaml:

xxxxxxxxxx

cp /opt/course/p1/project-23-api.yaml /opt/course/p1/project-23-api-new.yaml

vim /opt/course/p1/project-23-api-new.yaml

Add the liveness-probe to the yaml:

xxxxxxxxxx

\# /opt/course/p1/project-23-api-new.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: project-23-api

  namespace: pluto

spec:

  replicas: 3

  selector:

    matchLabels:

      app: project-23-api

  template:

    metadata:

      labels:

        app: project-23-api

    spec:

      volumes:

      - name: cache-volume1

        emptyDir: {}

      - name: cache-volume2

        emptyDir: {}

      - name: cache-volume3

        emptyDir: {}

      containers:

      - image: httpd:2.4-alpine

        name: httpd

        volumeMounts:

        - mountPath: /cache1

          name: cache-volume1

        - mountPath: /cache2

          name: cache-volume2

        - mountPath: /cache3

          name: cache-volume3

        env:

        - name: APP\_ENV

          value: "prod"

        - name: APP\_SECRET\_N1

          value: "IO=a4L/XkRdvN8jM=Y+"

        - name: APP\_SECRET\_P1

          value: "-7PA0\_Z\]>{pwa43r)\_\_"

        livenessProbe:                  \# add

          tcpSocket:                    \# add

            port: 80  \# add

          initialDelaySeconds: 10 \# add

          periodSeconds: 15 \# add

Then let's apply the changes:

xxxxxxxxxx

k \-f /opt/course/p1/project-23-api-new.yaml apply 

Next we wait 10 seconds and confirm the _Pods_ are still running:

xxxxxxxxxx

➜ k -n pluto get pod

NAME                              READY   STATUS    RESTARTS   AGE

holy-api                          1/1     Running   0          144m

project-23-api-5b4579fd49-8knh8   1/1     Running   0          90s

project-23-api-5b4579fd49-cbgph   1/1     Running   0          88s

project-23-api-5b4579fd49-tcfq5   1/1     Running   0          86s

We can also check the configured liveness-probe settings on a _Pod_ or the _Deployment_:

xxxxxxxxxx

➜ k -n pluto describe pod project-23-api-5b4579fd49-8knh8 | grep Liveness

    Liveness:   tcp-socket :80 delay=10s timeout=1s period=15s #success=1 #failure=3

➜  k -n pluto describe deploy project-23-api | grep Liveness

    Liveness:   tcp-socket :80 delay=10s timeout=1s period=15s #success=1 #failure=3

Preview Question 2
------------------

Solve this question on instance: `ssh ckad9043`

Team Sun needs a new _Deployment_ named `sunny` with 4 replicas of image `nginx:1.17.3-alpine` in _Namespace_ `sun`. The _Deployment_ and its _Pods_ should use the existing _ServiceAccount_ `sa-sun-deploy`.

Expose the _Deployment_ internally using a ClusterIP _Service_ named `sun-srv` on port 9999. The nginx containers should run as default on port 80. The management of Team Sun would like to execute a command to check that all _Pods_ are running on occasion. Write that command into file `/opt/course/p2/sunny_status_command.sh`. The command should use `kubectl`.

##### Answer

xxxxxxxxxx

k \-n sun create deployment \-h #help

k \-n sun create deployment sunny \--image\=nginx:1.17.3-alpine \--dry-run\=client \-oyaml > p2\_sunny.yaml

vim p2\_sunny.yaml

Then alter its yaml to include the requirements:

xxxxxxxxxx

\# p2\_sunny.yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  creationTimestamp: null

  labels:

    app: sunny

  name: sunny

  namespace: sun

spec:

  replicas: 4 \# change

  selector:

    matchLabels:

      app: sunny

  strategy: {}

  template:

    metadata:

      creationTimestamp: null

      labels:

        app: sunny

    spec:

      serviceAccountName: sa-sun-deploy \# add

      containers:

      - image: nginx:1.17.3-alpine

        name: nginx

        resources: {}

status: {}

Now create the yaml and confirm it's running:

xxxxxxxxxx

➜ k create -f p2\_sunny.yaml 

deployment.apps/sunny created

➜ k -n sun get pod

NAME                     READY   STATUS        RESTARTS   AGE

0509649a                 1/1     Running       0          149m

0509649b                 1/1     Running       0          149m

1428721e                 1/1     Running       0          149m

...

sunny-64df8dbdbb-9mxbw   1/1     Running       0          10s

sunny-64df8dbdbb-mp5cf   1/1     Running       0          10s

sunny-64df8dbdbb-pggdf   1/1     Running       0          6s

sunny-64df8dbdbb-zvqth   1/1     Running       0          7s

Confirmed, the AGE column is always in important information about if changes were applied. Next we expose the _Pods_ by created the _Service_:

xxxxxxxxxx

k \-n sun expose \-h \# help

k \-n sun expose deployment sunny \--name sun-srv \--port 9999 \--target-port 80

Using expose instead of `kubectl create service clusterip` is faster because it already sets the correct selector-labels. The previous command would produce this yaml:

xxxxxxxxxx

\# k -n sun expose deployment sunny --name sun-srv --port 9999 --target-port 80

apiVersion: v1

kind: Service

metadata:

  creationTimestamp: null

  labels:

    app: sunny

  name: sun-srv \# required by task

spec:

  ports:

  - port: 9999  \# service port

    protocol: TCP

    targetPort: 80  \# target port

  selector:

    app: sunny  \# selector is important

status:

  loadBalancer: {}

Let's test the _Service_ using `wget` from a temporary _Pod_:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 sun-srv.sun:9999

Connecting to sun-srv.sun:9999 (10.23.253.120:9999)

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

...

Because the _Service_ is in a different _Namespace_ as our temporary _Pod_, it is reachable using the names `sun-srv.sun` or fully: `sun-srv.sun.svc.cluster.local`.

Finally we need a command which can be executed to check if all _Pods_ are runing, this can be done with:

xxxxxxxxxx

vim /opt/course/p2/sunny\_status\_command.sh

xxxxxxxxxx

\# /opt/course/p2/sunny\_status\_command.sh

kubectl -n sun get deployment sunny

To run the command:

xxxxxxxxxx

➜ sh /opt/course/p2/sunny\_status\_command.sh

NAME    READY   UP-TO-DATE   AVAILABLE   AGE

sunny   4/4     4            4           13m

Preview Question 3
------------------

Solve this question on instance: `ssh ckad5601`

Management of EarthAG recorded that one of their _Services_ stopped working. Dirk, the administrator, left already for the long weekend. All the information they could give you is that it was located in _Namespace_ `earth` and that it stopped working after the latest rollout. All _Services_ of EarthAG should be reachable from inside the cluster.

Find the _Service_, fix any issues and confirm it's working again. Write the reason of the error into file `/opt/course/p3/ticket-654.txt` so Dirk knows what the issue was.

##### Answer

First we get an overview of the resources in _Namespace_ `earth`:

xxxxxxxxxx

➜ k -n earth get all

NAME                                          READY   STATUS    RESTARTS   AGE

pod/earth-2x3-api-584df69757-ngnwp            1/1     Running   0          116m

pod/earth-2x3-api-584df69757-ps8cs            1/1     Running   0          116m

pod/earth-2x3-api-584df69757-ww9q8            1/1     Running   0          116m

pod/earth-2x3-web-85c5b7986c-48vjt            1/1     Running   0          116m

pod/earth-2x3-web-85c5b7986c-6mqmb            1/1     Running   0          116m

pod/earth-2x3-web-85c5b7986c-6vjll            1/1     Running   0          116m

pod/earth-2x3-web-85c5b7986c-fnkbp            1/1     Running   0          116m

pod/earth-2x3-web-85c5b7986c-pjm5m            1/1     Running   0          116m

pod/earth-2x3-web-85c5b7986c-pwfvj            1/1     Running   0          116m

pod/earth-3cc-runner-6cb6cc6974-8wm5x         1/1     Running   0          116m

pod/earth-3cc-runner-6cb6cc6974-9fx8b         1/1     Running   0          116m

pod/earth-3cc-runner-6cb6cc6974-b9nrv         1/1     Running   0          116m

pod/earth-3cc-runner-heavy-6bf876f46d-b47vq   1/1     Running   0          116m

pod/earth-3cc-runner-heavy-6bf876f46d-mrzqd   1/1     Running   0          116m

pod/earth-3cc-runner-heavy-6bf876f46d-qkd74   1/1     Running   0          116m

pod/earth-3cc-web-6bfdf8b848-f74cj            0/1     Running   0          116m

pod/earth-3cc-web-6bfdf8b848-n4z7z            0/1     Running   0          116m

pod/earth-3cc-web-6bfdf8b848-rcmxs            0/1     Running   0          116m

pod/earth-3cc-web-6bfdf8b848-xl467            0/1     Running   0          116m

NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE

service/earth-2x3-api-svc   ClusterIP   10.3.241.242   <none>        4546/TCP   116m

service/earth-2x3-web-svc   ClusterIP   10.3.250.247   <none>        4545/TCP   116m

service/earth-3cc-web       ClusterIP   10.3.243.24    <none>        6363/TCP   116m

NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE

deployment.apps/earth-2x3-api            3/3     3            3           116m

deployment.apps/earth-2x3-web            6/6     6            6           116m

deployment.apps/earth-3cc-runner         3/3     3            3           116m

deployment.apps/earth-3cc-runner-heavy   3/3     3            3           116m

deployment.apps/earth-3cc-web            0/4     4            0           116m

NAME                                                DESIRED   CURRENT   READY   AGE

replicaset.apps/earth-2x3-api-584df69757            3         3         3       116m

replicaset.apps/earth-2x3-web-85c5b7986c            6         6         6       116m

replicaset.apps/earth-3cc-runner-6cb6cc6974         3         3         3       116m

replicaset.apps/earth-3cc-runner-heavy-6bf876f46d   3         3         3       116m

replicaset.apps/earth-3cc-web-6895587dc7            0         0         0       116m

replicaset.apps/earth-3cc-web-6bfdf8b848            4         4         0       116m

replicaset.apps/earth-3cc-web-d49645966             0         0         0       116m

First impression could be that all _Pods_ are in status RUNNING. But looking closely we see that some of the _Pods_ are not ready, which also confirms what we see about one _Deployment_ and one _replicaset_. This could be our error to further investigate.

Another approach could be to check the _Service_s for missing endpoints:

xxxxxxxxxx

➜ k -n earth get ep

NAME                ENDPOINTS                                           AGE

earth-2x3-api-svc   10.0.0.10:80,10.0.1.5:80,10.0.2.4:80                116m

earth-2x3-web-svc   10.0.0.11:80,10.0.0.12:80,10.0.1.6:80 + 3 more...   116m

earth-3cc-web           

_Service_ `earth-3cc-web` doesn't have endpoints. This could be a selector/label misconfiguration or the endpoints are actually not available/ready.

Checking all _Services_ for connectivity should show the same (this step is optional and just for demonstration):

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 earth-2x3-api-svc.earth:4546

...

<html><body><h1>It works!</h1></body></html>

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 earth-2x3-web-svc.earth:4545

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100    45  100    45    0     0   5000      0 --:--:-- --:--:-- --:--:--  5000

<html><body><h1>It works!</h1></body></html>

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 earth-3cc-web.earth:6363

If you don't see a command prompt, try pressing enter.

  0     0    0     0    0     0      0      0 --:--:--  0:00:05 --:--:--     0

curl: (28) Connection timed out after 5000 milliseconds

pod "tmp" deleted

pod default/tmp terminated (Error)

Notice that we use here for example `earth-2x3-api-svc.earth`. We could also spin up a temporary _Pod_ in _Namespace_ `earth` and connect directly to `earth-2x3-api-svc`.

We get no connection to `earth-3cc-web.earth:6363`. Let's look at the _Deployment_ `earth-3cc-web`. Here we see that the requested amount of replicas is not available/ready:

xxxxxxxxxx

➜ k -n earth get deploy earth-3cc-web

NAME            READY   UP-TO-DATE   AVAILABLE   AGE

earth-3cc-web   0/4     4            0           7m18s

To continue we check the _Deployment_ yaml for some misconfiguration:

xxxxxxxxxx

k \-n earth edit deploy earth-3cc-web

xxxxxxxxxx

\# k -n earth edit deploy earth-3cc-web

apiVersion: extensions/v1beta1

kind: Deployment

metadata:

...

  generation: 3 \# there have been rollouts

  name: earth-3cc-web

  namespace: earth

...

spec:

...

  template:

    metadata:

      creationTimestamp: null

      labels:

        id: earth-3cc-web

    spec:

      containers:

      - image: nginx:1.16.1-alpine

        imagePullPolicy: IfNotPresent

        name: nginx

        readinessProbe:

          failureThreshold: 3

          initialDelaySeconds: 10

          periodSeconds: 20

          successThreshold: 1

          tcpSocket:

            port: 82  \# this port doesn't seem to be right, should be 80

          timeoutSeconds: 1

...

We change the readiness-probe port, save and check the _Pods_:

xxxxxxxxxx

➜ k -n earth get pod -l id=earth-3cc-web

NAME                            READY   STATUS    RESTARTS   AGE

earth-3cc-web-d49645966-52vb9   0/1     Running   0          6s

earth-3cc-web-d49645966-5tts6   0/1     Running   0          6s

earth-3cc-web-d49645966-db5gp   0/1     Running   0          6s

earth-3cc-web-d49645966-mk7gr   0/1     Running   0          6s

Running, but still not in ready state. Wait 10 seconds (initialDelaySeconds of readinessProbe) and check again:

xxxxxxxxxx

➜ k -n earth get pod -l id=earth-3cc-web

NAME                            READY   STATUS    RESTARTS   AGE

earth-3cc-web-d49645966-52vb9   1/1     Running   0          32s

earth-3cc-web-d49645966-5tts6   1/1     Running   0          32s

earth-3cc-web-d49645966-db5gp   1/1     Running   0          32s

earth-3cc-web-d49645966-mk7gr   1/1     Running   0          32s

Let's check the service again:

xxxxxxxxxx

➜ k run tmp --restart=Never --rm -i --image=nginx:alpine -- curl -m 5 earth-3cc-web.earth:6363

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current

                                 Dload  Upload   Total   Spent    Left  Speed

100   612  100   612    0     0  55636      0 --:--:-- --:--:-- --:--:-- 55636

<!DOCTYPE html>

<html>

<head>

<title>Welcome to nginx!</title>

<style>

    body {

        width: 35em;

        margin: 0 auto;

        font-family: Tahoma, Verdana, Arial, sans-serif;

    }

</style>

</head>

<body>

<h1>Welcome to nginx!</h1>

...

We did it! Finally we write the reason into the requested location:

xxxxxxxxxx

vim /opt/course/p3/ticket-654.txt

xxxxxxxxxx

\# /opt/course/p3/ticket-654.txt

yo Dirk, wrong port for readinessProbe defined! 

