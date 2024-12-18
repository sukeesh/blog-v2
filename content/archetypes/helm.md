---
layout: post
author: "Sukeesh"
date: 2019-07-02
title: "Installing helm chart"
---

`$ helm init`

`$ helm create test-sukeesh`

This will create all the files necessary, We are only interested in `Chart.yaml`, `values.yaml` files.

`$ helm lint` this would lint all the files

Once you are done with all the changes, then go back to the root dir and do `helm install`

`$ cd ..`
`$ helm install test-sukeesh-0.1.0.tgz`

But, when you try above command, you might see some error related to invalid permissions.

So, when you try to tail the logs of tiller pod in `kube-system` namespace which is installed when we did `helm init`
We see these errors like these
```
tiller-deploy-c78f4c8d9-2q4jz tiller [tiller] 2019/07/02 07:56:00 preparing install for
tiller-deploy-c78f4c8d9-2q4jz tiller [storage] 2019/07/02 07:56:00 getting release "orbiting-hog.v1"
tiller-deploy-c78f4c8d9-2q4jz tiller [storage/driver] 2019/07/02 07:56:00 get: failed to get "orbiting-hog.v1": configmaps "orbiting-hog.v1" is forbidden: User "system:serviceaccount:kube-system:default" cannot get resource "configmaps" in API group "" in the namespace "kube-system"
```
So, by default when we don't specify a service account in the service, this uses `"system:serviceaccount:kube-system:default"` service account but then this has no permissions.
So, create a Role-based access control file
`rbac-config.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
Apply this `rbac-config.yaml` file

```
$ kubectl create -f rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created
```
You can see serviceaccount and clusterrolebinding "tiller" has been created successfully.
Verify with the following
```
$ kubectl get serviceaccounts | grep tiller
tiller                               1         30s
```
But, when you try installing the helm, you might again get the same error because we created a service account but did not associate it with our tiller deployment.

```
$ kubectl get deploy | grep tiller
tiller-deploy            1         1         1            1           91m
```

```
$ kubectl edit deploy tiller-deploy
deployment.extensions/tiller-deploy edited
```
Edit the `tiller-deploy` deployment and add serviceAccountName to spec.template.spec as below
```
...
serviceAccountName: tiller
...
```

Now, helm is able to install with no error

```
$ helm install test-sukeesh-0.1.0.tgz --debug

[debug] Created tunnel using local port: '52030'

[debug] SERVER: "127.0.0.1:52030"

[debug] Original chart version: ""
[debug] CHART PATH: /Users/sukeesh/Documents/helm/test-sukeesh-0.1.0.tgz

NAME:   alliterating-wildebeest
REVISION: 1
RELEASED: Tue Jul  2 21:27:15 2019
CHART: test-sukeesh-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
image:
  pullPolicy: IfNotPresent
  repository: daemonza/testapi
  tag: latest
ingress:
  annotations: {}
  enabled: false
  hosts:
  - chart-example.local
  path: /
  tls: []
nodeSelector: {}
replicaCount: 2
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi
service:
  externalPort: 80
  internalPort: 80
  name: test-sukeesh
  port: 9000
  type: ClusterIP
tolerations: []

HOOKS:
MANIFEST:

---
# Source: test-sukeesh/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: alliterating-wildebeest-test-sukeesh
  labels:
    app: test-sukeesh
    chart: test-sukeesh-0.1.0
    release: alliterating-wildebeest
    heritage: Tiller
spec:
  type: ClusterIP
  ports:
    - port: 9000
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: test-sukeesh
    release: alliterating-wildebeest
---
# Source: test-sukeesh/templates/deployment.yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: alliterating-wildebeest-test-sukeesh
  labels:
    app: test-sukeesh
    chart: test-sukeesh-0.1.0
    release: alliterating-wildebeest
    heritage: Tiller
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test-sukeesh
      release: alliterating-wildebeest
  template:
    metadata:
      labels:
        app: test-sukeesh
        release: alliterating-wildebeest
    spec:
      containers:
        - name: test-sukeesh
          image: "daemonza/testapi:latest"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 100m
              memory: 128Mi
LAST DEPLOYED: Tue Jul  2 21:27:15 2019
NAMESPACE: kube-system
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                                  TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
alliterating-wildebeest-test-sukeesh  ClusterIP  10.59.245.130  <none>       9000/TCP  1s

==> v1beta2/Deployment
NAME                                  DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
alliterating-wildebeest-test-sukeesh  2        2        2           0          0s

==> v1/Pod(related)
NAME                                                   READY  STATUS             RESTARTS  AGE
alliterating-wildebeest-test-sukeesh-6d577d4888-bftnq  0/1    ContainerCreating  0         0s
alliterating-wildebeest-test-sukeesh-6d577d4888-ntmzz  0/1    ContainerCreating  0         0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace kube-system -l "app=test-sukeesh,release=alliterating-wildebeest" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```