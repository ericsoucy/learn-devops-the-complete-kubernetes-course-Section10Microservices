# Learn DevOps: The Complete Kubernetes Course

## Section 10 Microservices

### Ch 120 Introduction istio

![micro services](micro-services.png)
![micro services side cars](micro-services2.png)
![istio](istio.png)

```bash
## not working on arm64 : FAIL
 curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.9.1 TARGET_ARCH=arm64 sh -
 sudo ./istio-1.9.1/bin/istioctl install --set profile=demo -y --kubeconfig /etc/rancher/k3s/k3s.yaml
```

```bash
## on docker desktop wsl2
curl -L https://istio.io/downloadIstio | sh -
istio-1.9.1/bin/istioctl install --set profile=demo -y
kubectl label namespace istio-tests istio-injection=enabled
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -n istio-tests
kubectl get services -n istio-tests
kubectl get pods -n istio-tests

kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}' -n istio-tests)" -c ratings -n istio-tests -- curl -sS productpage:9080/productpage  | grep -o "<title>.*</title>"

kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml -n istio-tests

istioctl analyze -n istio-tests

kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}'
//192.168.65.3
kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}'
//30498

curl -vvvv http://kubernetes.docker.internal/productpage
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml -n istio-tests

//build http-echo
docker build -t ericsoucy/http-echo .
docker run  -e TEXT=hello -p 8080:8080 ericsoucy/http-echo
docker push ericsoucy/http-echo
```

### Ch 122 Demo: An Istio enabled app

```bash
wget https://raw.githubusercontent.com/ericsoucy/learn-devops-the-complete-kubernetes-course-Section10Microservices/main/istio/helloworld.yaml

kubectl apply -f <(./istio-1.9.1/bin/istioctl kube-inject -f helloworld.yaml -n istio-tests) -n istio-tests
kubectl get pods -n istio-tests

kubectl apply -f https://raw.githubusercontent.com/ericsoucy/learn-devops-the-complete-kubernetes-course-Section10Microservices/main/istio/helloworld-gw.yaml -n istio-tests

curl -vvvv http://kubernetes.docker.internal/hello
```

### ch 123 Demo: Advanced routing with Istio

![istio](istio-hello-world-v2.png)

```bash
wget https://raw.githubusercontent.com/ericsoucy/learn-devops-the-complete-kubernetes-course-Section10Microservices/main/istio/helloworld-v2.yaml

kubectl apply -f <(./istio-1.9.1/bin/istioctl kube-inject -f helloworld-v2.yaml -n istio-tests) -n istio-tests
kubectl get pods -n istio-tests

kubectl apply -f helloworld-v2-routing.yaml -n istio-tests

curl -vvvv  -H "host: hello.example.com" http://kubernetes.docker.internal/hello

curl -vvvv  -H "host: hello.example.com" -H "end-user: john" http://kubernetes.docker.internal/hello

```

### ch 124. Demo: Canary Deployments

![canary](canary.png)

```bash
kubectl apply -f helloworld-v2-canary.yaml -n istio-tests

kubectl describe virtualservice helloworld -n istio-tests

for ((i=1;i<=10;i++)); do curl -H "host: hello.example.com" http://kubernetes.docker.internal/hello; done
```

### ch 125. Demo: Retries

![retries](./retries.png)

```bash
kubectl apply -f helloworld-v3.yaml -n istio-tests

curl -vvvv  -H "host: hello-v3.example.com" http://kubernetes.docker.internal/hello

for ((i=1;i<=10;i++)); do time curl -H "host: hello-v3.example.com" http://kubernetes.docker.internal/hello; done


// comment retries
kubectl apply -f helloworld-v3.yaml -n istio-tests

```

### ch 126. Mutual TLS

<https://istio.io/latest/docs/concepts/security/#mutual-tls-authentication>

![security](https://istio.io/latest/docs/concepts/security/arch-sec.svg)

![security](https://istio.io/latest/docs/concepts/security/id-prov.svg)

![auth](https://istio.io/latest/docs/concepts/security/authn.svg)

![mutual-tls](./mutual-tls.png)

![demo](./demo-tls.png)

```bash
wget https://raw.githubusercontent.com/ericsoucy/learn-devops-the-complete-kubernetes-course-Section10Microservices/main/istio/helloworld-tls.yaml


kubectl apply -f <(./istio-1.9.1/bin/istioctl kube-inject -f helloworld-tls.yaml)
kubectl apply -f helloworld-tls-legacy.yaml

kubectl get svc -o wide -n istio-system

curl -vvvv  -H "host: hello-tls.example.com" http://kubernetes.docker.internal/
```