# Istio Ingress Gateway Deployment on Different Namespaces and Nodes

This lab demonstrates the process of deploying Istio Ingress Gateways across different namespaces and nodes in a Kubernetes cluster. Key components of the lab include:

- Setting up multiple Istio versions (1.16.7 and 1.17.5) in the cluster
- Configuring separate ingress gateways for partner, security, and external traffic
- Creating and applying YAML configurations for different ingress gateways
- Executing deployment commands to set up the Istio control plane and ingress gateways
- Implementing node affinity to control pod placement across worker nodes
- Performing verification steps to ensure proper deployment and functioning of the ingress gateways

The lab provides hands-on experience in managing a complex Istio environment, showcasing how to handle traffic routing and pod placement in a Kubernetes cluster with multiple ingress gateways.

![](imgs/multiple_ingress_gateways.png)


Ref: 

https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/#GatewaySpec

https://istio.io/latest/docs/setup/upgrade/canary/

https://istio.io/v1.16/blog/2021/revision-tags/


Manually install ISTIO version 1-16-7 and 1-17-5
---

```
kubectl create ns istio-system

kubectl apply -f istiod-service.yaml -n istio-system

asdf global istioctl 1.16.7

export ISTIO_REVISION=1-16-7

istioctl install -y -n istio-system -f istiod-controlplane.yaml --revision ${ISTIO_REVISION}

asdf global istioctl 1.17.5

export ISTIO_REVISION=1-17.5

istioctl install -y -n istio-system -f istiod-controlplane.yaml --revision ${ISTIO_REVISION}

kubectl get iop -A
```

```
kubectl get all -n istio-system

NAME                                 READY   STATUS    RESTARTS   AGE
pod/istiod-1-16-7-5d6d495946-9njqj   1/1     Running   0          28m
pod/istiod-1-17-5-65fdb4d666-qrst4   1/1     Running   0          24m

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                 AGE
service/istiod          ClusterIP   10.123.91.125    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   29m
service/istiod-1-16-7   ClusterIP   10.123.161.85    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   28m
service/istiod-1-17-5   ClusterIP   10.123.230.132   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   24m

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istiod-1-16-7   1/1     1            1           28m
deployment.apps/istiod-1-17-5   1/1     1            1           24m

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/istiod-1-16-7-5d6d495946   1         1         1       28m
replicaset.apps/istiod-1-17-5-65fdb4d666   1         1         1       24m

NAME                                                REFERENCE                  TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istiod-1-16-7   Deployment/istiod-1-16-7   <unknown>/80%   1         5         1          28m
horizontalpodautoscaler.autoscaling/istiod-1-17-5   Deployment/istiod-1-17-5   <unknown>/80%   1         5         1          24m

```


Deploying ISTIOD 1.16.7 and Ingress Gateways 1.16.7
---

```
istioctl version
#check current selected version
asdf list
asdf global istioctl 1.16.7
env | grep ISTIO
export ISTIO_REVISION=1-16-7

kubectl create ns partner-igw-1-16-7
kubectl create ns security-igw-1-16-7
kubectl create ns external-igw-1-16-7

watch kubectl get iop -A
watch kubectl get all -n istio-system
watch kubectl get all -n partner-igw-1-16-7 
watch kubectl get all -n external-igw-1-16-7 
watch kubectl get all -n security-igw-1-16-7 

istioctl install -y -n partner-igw-1-16-7 -f partner-igw-1-16-7.yaml --revision ${ISTIO_REVISION}
istioctl install -y -n external-igw-1-16-7 -f external-igw-1-16-7.yaml --revision ${ISTIO_REVISION}
istioctl install -y -n security-igw-1-16-7 -f security-igw-1-16-7.yaml --revision ${ISTIO_REVISION}
```

Deployment for ISTIOD 1.17.5 and Ingress Gateways 1.17.5
---
```
istioctl version
#check current selected version
asdf list
asdf global istioctl 1.17.5
env | grep ISTIO
export ISTIO_REVISION=1-17-5

kubectl create ns partner-igw-1-17-5
kubectl create ns security-igw-1-17-5
kubectl create ns external-igw-1-17-5

watch kubectl get iop -A
watch kubectl get all -n istio-system
watch kubectl get all -n partner-igw-1-17-5 
watch kubectl get all -n external-igw-1-17-5 
watch kubectl get all -n security-igw-1-17-5

istioctl install -y -n partner-igw-1-17-5 -f partner-igw-1-17-5.yaml --revision ${ISTIO_REVISION}
istioctl install -y -n external-igw-1-17-5 -f external-igw-1-17-5.yaml --revision ${ISTIO_REVISION}
istioctl install -y -n security-igw-1-17-5 -f security-igw-1-17-5.yaml --revision ${ISTIO_REVISION}
```

```
kubectl get iop -A

NAMESPACE      NAME                                                              REVISION   STATUS   AGE
istio-system   installed-state-control-plane-1-16-7                              1-16-7              35m
istio-system   installed-state-control-plane-1-17-5                              1-17-5              47m
istio-system   installed-state-istio-ingress-gw-install-external-1-16-7-1-16-7   1-16-7              59s
istio-system   installed-state-istio-ingress-gw-install-external-1-17-5          1-17-5              6m43s
istio-system   installed-state-istio-ingress-gw-install-partner-1-16-7-1-16-7    1-16-7              2m4s
istio-system   installed-state-istio-ingress-gw-install-partner-1-17-5           1-17-5              7m38s
istio-system   installed-state-istio-ingress-gw-install-security-1-16-7-1-16-7   1-16-7              9s
istio-system   installed-state-istio-ingress-gw-install-security-1-17-5          1-17-5              3m12s
```

Deploying Gateway and Virtual Service
---

In this lab setup, incoming traffic for the host ***hellocloud.io*** is directed through the ***partner-igw-1-16-7*** ingress gateway. To achieve this, the selector gw: partner-igw-1-16-7 is applied, ensuring that traffic specifically routes through this gateway.

From there, the traffic is forwarded to the relevant workloads, which are managed by the ***ISTIOD revision 1-16-7***. This configuration ensures both that traffic flows through the specified gateway and that it is handled by workloads controlled by the desired Istio control plane revision.

ingress/web-api-gw.yaml
  ```
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: web-api-gateway
spec:
  selector:
    gw: partner-igw-1-16-7 
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "hellocloud.io"
  ```
ingress/web-api-vs.yaml

```
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: web-api-gw-vs
spec:
  hosts:
  - "hellocloud.io"
  gateways:
  - web-api-gateway
  http:
  - route:
    - destination:
        host: web-api.hellocloud.svc.cluster.local # fix it
        port:
          number: 8080
```

In other cases, we can configure different ingress gateways versions for different incoming traffice and hosts, depending on the scenarios.
 

```
kubectl apply -f ./ingress/web-api-gw.yaml -n hellocloud
kubectl apply -f ./ingress/web-api-vs.yaml -n hellocloud

istioctl pc listener deployment/partner-igw-1-16-7 -n partner-igw-1-16-7

ADDRESS PORT  MATCH DESTINATION
0.0.0.0 8080  ALL   Route: http.8080
0.0.0.0 15021 ALL   Inline Route: /healthz/ready*
0.0.0.0 15090 ALL   Inline Route: /stats/prometheus*

istioctl pc routes deployment/partner-igw-1-16-7 -n partner-igw-1-16-7

NAME          DOMAINS           MATCH                  VIRTUAL SERVICE
http.8080     hellocloud.io     /*                     web-api-gw-vs.hellocloud
              *                 /healthz/ready*        
              *                 /stats/prometheus*     

```



Verifying the ingress gateway (partner-igw-1-16-7)
---

Create the *hellocloud namespace*.

Enalbe Istio Injection on the namespace.

Label the namespace with Istio revision 1-16-7 so that the istio-proxy sidecar containers in workloads within this namespace will be managed by the ISTIOD control plane revision 1-16-7

```
kubectl get ns
kubectl create ns hellocloud

kubectl label ns hellocloud istio-injection=enabled

kubectl get ns -n hellocloud --show-labels

export ISTIO_REVISION=1-16-7
echo $ISTIO_REVISION

kubectl label namespace hellocloud istio.io/rev=${ISTIO_REVISION}

kubectl get ns -n hellocloud --show-labels

hellocloud            Active   148m   istio-injection=enabled,istio.io/rev=1-16-7,kubernetes.io/metadata.name=hellocloud
```


Deploy sample apps to the hellocloud namespace.

```
kubectl apply -f ../sample-apps/ -n hellocloud
```

Deploy Gateway and Virtual service.

```
#install web-api-gw, web-api-vs
kubectl apply -f ../ingress/ -n hellocloud
```

Verify that labels are correct and all resources up and running

```
#ISTIO is injected and will use the revision 1-16-7
kubectl get ns hellocloud --show-labels

NAME         STATUS   AGE   LABELS
hellocloud   Active   26m   istio-injection=enabled,istio.io/rev=1-16-7,kubernetes.io/metadata.name=hellocloud


#Verifying all resources up and running
kubectl get all -n hellocloud

NAME                                       READY   STATUS    RESTARTS   AGE
pod/purchase-history-v1-764c68bcf5-nxpm5   2/2     Running   0          27m
pod/recommendation-5b8dcb868-f5npd         2/2     Running   0          27m
pod/sleep-847976dfd9-jxjvs                 2/2     Running   0          27m
pod/web-api-64bf767798-5f7h7               2/2     Running   0          27m

NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/purchase-history   ClusterIP   10.123.69.101    <none>        8080/TCP   27m
service/recommendation     ClusterIP   10.123.105.10    <none>        8080/TCP   27m
service/sleep              ClusterIP   10.123.134.79    <none>        80/TCP     27m
service/web-api            ClusterIP   10.123.198.149   <none>        8080/TCP   27m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/purchase-history-v1   1/1     1            1           27m
deployment.apps/recommendation        1/1     1            1           27m
deployment.apps/sleep                 1/1     1            1           27m
deployment.apps/web-api               1/1     1            1           27m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/purchase-history-v1-764c68bcf5   1         1         1       27m
replicaset.apps/recommendation-5b8dcb868         1         1         1       27m
replicaset.apps/sleep-847976dfd9                 1         1         1       27m
replicaset.apps/web-api-64bf767798               1         1         1       27m
```


Check if the pods are using the ISTIO REVISION 1-16-7

```
kubectl describe pod web-api-64bf767798-5f7h7 -n hellocloud | grep -i "image"

    Image:         docker.io/istio/proxyv2:1.16.7
    Image ID:      docker.io/istio/proxyv2@sha256:6cccf60e48ca03a2ad12a20a2f4d63ab442c32778cfc473906906452b4dc6170
    Image:          nicholasjackson/fake-service:v0.7.8
    Image ID:       docker.io/nicholasjackson/fake-service@sha256:614f71035f1adf4d94b189a3e0cc7b49fe783cf97b6b00b5e24f3c235f4ea14e
    Image:         docker.io/istio/proxyv2:1.16.7
    Image ID:      docker.io/istio/proxyv2@sha256:6cccf60e48ca03a2ad12a20a2f4d63ab442c32778cfc473906906452b4dc6170
  Normal   Pulling    10m                kubelet            Pulling image "docker.io/istio/proxyv2:1.16.7"
  Normal   Pulled     10m                kubelet            Successfully pulled image "docker.io/istio/proxyv2:1.16.7" in 10.970923574s
  Normal   Pulling    10m                kubelet            Pulling image "nicholasjackson/fake-service:v0.7.8"
  Normal   Pulled     10m                kubelet            Successfully pulled image "nicholasjackson/fake-service:v0.7.8" in 6.681423087s
  Normal   Pulled     10m                kubelet            Container image "docker.io/istio/proxyv2:1.16.7" already present on machine

```

Verifying all ingress gateways working or not in the namesapces

```

kubectl get all -n partner-igw-1-16-7
kubectl get all -n partner-igw-1-16-7
kubectl get all -n partner-igw-1-16-7

export GATEWAY_IP=$(kubectl get svc -n partner-igw-1-16-7 partner-igw-1-16-7 -o jsonpath="{.status.loadBalancer.ingress[0].ip}")

export INGRESS_PORT=80

export SECURE_INGRESS_PORT=443

```

Finally, the incoming traffic for the host "hellocloud.io" goes through the ingress gateway partner-igw-1-16-7

```
curl -H "Host: hellocloud.io" http://$GATEWAY_IP:$INGRESS_PORT

{
  "name": "web-api",
  "uri": "/",
  "type": "HTTP",
  "ip_addresses": [
    "10.243.3.4"
  ],
  "start_time": "2024-11-03T06:31:17.585141",
  "end_time": "2024-11-03T06:31:17.607570",
  "duration": "22.428476ms",
  "body": "Hello From Web API",
  "upstream_calls": [
    {
      "name": "recommendation",
      "uri": "http://recommendation:8080",
      "type": "HTTP",
      "ip_addresses": [
        "10.243.1.11"
      ],
      "start_time": "2024-11-03T06:31:17.587761",
      "end_time": "2024-11-03T06:31:17.606348",
      "duration": "18.586931ms",
      "body": "Hello From Recommendations!",
      "upstream_calls": [
        {
          "name": "purchase-history-v1",
          "uri": "http://purchase-history:8080",
          "type": "HTTP",
          "ip_addresses": [
            "10.243.1.10"
          ],
          "start_time": "2024-11-03T06:31:17.590701",
          "end_time": "2024-11-03T06:31:17.590801",
          "duration": "100.258µs",
          "body": "Hello From Purchase History (v1)!",
          "code": 200
        }
      ],
      "code": 200
    }
  ],
  "code": 200

```  



To schedule Ingress Gateway Pods on Different Nodes (Use Affinity)
---

The following YAML snippet is added to the ingress gateway YAML files:

```
affinity: 
          nodeAffinity: 
            requiredDuringSchedulingIgnoredDuringExecution: 
              nodeSelectorTerms: 
              - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: In
                  values:
                  - 123-worker
```


This YAML snippet represents Kubernetes Node Affinity configuration, specifically using `requiredDuringSchedulingIgnoredDuringExecution`. It's a way to specify rules about the scheduling of pods onto nodes based on node labels.

Let's break down what this snippet means:

- `affinity:`: This indicates that you're configuring affinity rules for pod scheduling.
- `nodeAffinity:`: Specifies that the affinity rule is related to nodes.

Then you have:

- `requiredDuringSchedulingIgnoredDuringExecution:`: This section specifies that the rules must be met for pod scheduling and must be adhered to while the pod is running.
- `nodeSelectorTerms:`: This is a list of node selection criteria.
    - `matchExpressions:`: Rules that must be satisfied for the pod to be scheduled.
        - `key: kubernetes.io/hostname`: The node label key. This rule looks at the `kubernetes.io/hostname` label.
        - `operator: In`: The operator used to match the value. In this case, it's checking if the value is within the provided list.
        - `values:`: The list of values that the `kubernetes.io/hostname` label should match for pod scheduling. In this case, it's expecting nodes with the hostname `123-worker`.

This configuration will ensure that pods with this affinity rule will only be scheduled onto nodes that have the label `kubernetes.io/hostname` set to `123-worker`.

```
kubectl get nodes -A --show-labels
NAME                STATUS   ROLES                  AGE   VERSION    LABELS
123-control-plane   Ready    control-plane,master   66m   v1.23.10   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=123-control-plane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
123-worker          Ready    <none>                 66m   v1.23.10   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=123-worker,kubernetes.io/os=linux
123-worker2         Ready    <none>                 65m   v1.23.10   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=123-worker2,kubernetes.io/os=linux
123-worker3         Ready    <none>                 66m   v1.23.10   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=123-worker3,kubernetes.io/os=linux
```


Apply YAML files in the affinity directory, all ingress gateways are configured to run in the respective namespaces.

```
istioctl install -f ./affinity/ 
```

Verfifying

```
vagrant@istiocluster-box:~/istiocluster-box$ kubectl get all -A -o wide
NAMESPACE             NAME                                            READY   STATUS            RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
external-igw-1-16-7   pod/external-igw-1-16-7-745b47f6cc-q8248        1/1     Running           0          3m23s   10.243.1.4   123-worker3         <none>           <none>
external-igw-1-17-5   pod/external-igw-1-17-5-5d8475db5b-kp2bb        1/1     Running           0          9m23s   10.243.1.3   123-worker3         <none>           <none>
hellocloud            pod/purchase-history-v1-5f85bb478-nrthr         0/2     Init:0/1          0          40s     <none>       123-worker3         <none>           <none>
hellocloud            pod/purchase-history-v1-764c68bcf5-8g7mm        1/1     Running           0          26m     10.243.3.6   123-worker2         <none>           <none>
hellocloud            pod/recommendation-5b8dcb868-ppx2g              1/1     Running           0          26m     10.243.3.3   123-worker2         <none>           <none>
hellocloud            pod/recommendation-5ffb45bbc5-t9pkg             0/2     Init:0/1          0          40s     <none>       123-worker2         <none>           <none>
hellocloud            pod/sleep-5b4cd5667b-x2mgm                      0/2     PodInitializing   0          40s     10.243.2.5   123-worker          <none>           <none>
hellocloud            pod/sleep-847976dfd9-bj9nt                      1/1     Running           0          26m     10.243.3.4   123-worker2         <none>           <none>
hellocloud            pod/web-api-64bf767798-rvkwc                    1/1     Running           0          26m     10.243.3.5   123-worker2         <none>           <none>
hellocloud            pod/web-api-67768c496c-mzbbw                    0/2     Init:0/1          0          40s     <none>       123-worker2         <none>           <none>
istio-system          pod/istiod-1-16-7-5d6d495946-lzv2l              1/1     Running           0          38m     10.243.2.2   123-worker          <none>           <none>
istio-system          pod/istiod-1-17-5-65fdb4d666-s6jwr              1/1     Running           0          49m     10.243.1.2   123-worker3         <none>           <none>
kube-system           pod/coredns-64897985d-m59p2                     1/1     Running           0          62m     10.243.0.2   123-control-plane   <none>           <none>
kube-system           pod/coredns-64897985d-vpf7t                     1/1     Running           0          62m     10.243.0.3   123-control-plane   <none>           <none>
kube-system           pod/etcd-123-control-plane                      1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
kube-system           pod/kindnet-6zd6b                               1/1     Running           0          62m     172.18.0.4   123-worker2         <none>           <none>
kube-system           pod/kindnet-gdhtd                               1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
kube-system           pod/kindnet-gqkwp                               1/1     Running           0          62m     172.18.0.5   123-worker          <none>           <none>
kube-system           pod/kindnet-q6stg                               1/1     Running           0          62m     172.18.0.3   123-worker3         <none>           <none>
kube-system           pod/kube-apiserver-123-control-plane            1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
kube-system           pod/kube-controller-manager-123-control-plane   1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
kube-system           pod/kube-proxy-l4mc4                            1/1     Running           0          62m     172.18.0.4   123-worker2         <none>           <none>
kube-system           pod/kube-proxy-m78qt                            1/1     Running           0          62m     172.18.0.5   123-worker          <none>           <none>
kube-system           pod/kube-proxy-mzjd7                            1/1     Running           0          62m     172.18.0.3   123-worker3         <none>           <none>
kube-system           pod/kube-proxy-w4hpx                            1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
kube-system           pod/kube-scheduler-123-control-plane            1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
local-path-storage    pod/local-path-provisioner-58dc9cd8d9-z25f9     1/1     Running           0          62m     10.243.0.4   123-control-plane   <none>           <none>
metallb-system        pod/controller-7967ffcf8-wsjls                  1/1     Running           0          62m     10.243.3.2   123-worker2         <none>           <none>
metallb-system        pod/speaker-768qx                               1/1     Running           0          61m     172.18.0.5   123-worker          <none>           <none>
metallb-system        pod/speaker-96b9b                               1/1     Running           0          62m     172.18.0.3   123-worker3         <none>           <none>
metallb-system        pod/speaker-fjppp                               1/1     Running           0          62m     172.18.0.2   123-control-plane   <none>           <none>
metallb-system        pod/speaker-nts9v                               1/1     Running           0          62m     172.18.0.4   123-worker2         <none>           <none>
partner-igw-1-16-7    pod/partner-igw-1-16-7-5fbd96d6ff-xcjdf         1/1     Running           0          4m28s   10.243.2.4   123-worker          <none>           <none>
partner-igw-1-17-5    pod/partner-igw-1-17-5-ffcbc87bd-4fz8b          1/1     Running           0          10m     10.243.2.3   123-worker          <none>           <none>
security-igw-1-16-7   pod/security-igw-1-16-7-5d598c5969-qw4jj        1/1     Running           0          2m33s   10.243.3.8   123-worker2         <none>           <none>
security-igw-1-17-5   pod/security-igw-1-17-5-b9c6c8f48-vbq4n         1/1     Running           0          6m7s    10.243.3.7   123-worker2         <none>           <none>
```
