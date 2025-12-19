### Istio Service Mesh

## Monolitics to Microservice 

`Monolithic Architecture Challenges:`
- Example: Bookinfo Application
- Modules: Details, Reviews, Ratings, Product Page
- Issues:
  1. Tight coupling and shared database
  2. Scalability and independent updates are difficult
  3. Introducing new features or languages is cumbersome
  4. Risk of evolving into a "big ball of mud"
  
`Microservices Transformation:`
- Bookinfo Reimagined:
  Product Page → Python
  Details → Ruby
  Reviews → Java (with multiple versions)
  Ratings → Node.js
- Benefits:
  1. Independent scaling and faster releases
  2. Technological flexibility
  3. Enhanced resilience and manageability
  
`Microservices Challenges:`
- Cross-cutting concerns (e.g., networking, auth, logging) become decentralized
- Increased operational complexity
- Need for robust observability and collaboration (DevOps)

`Next Steps:`
- Introduction to Service Mesh as a solution to manage microservices complexity
- Tools like Istio help with traffic control, security, and observability

## Service Mesh 

- `Definition:` A service mesh is a configurable infrastructure layer that manages service-to-service communication in microservices architectures without altering business logic.
- Traditional Microservices: Each service handled its own routing, security, and observability.
- Service Mesh Approach:
  1. Uses sidecar proxies (like Envoy) alongside each microservice.
  2. These proxies form the data plane.
  3. A central Control Plane manages traffic and configurations
- Key Benefits of a Service Mesh:
  1. Dynamic Configuration: Change service interactions without modifying code.
  2. Enhanced Security: Mutual TLS for secure service communication.
  3. Comprehensive Observability: Real-time monitoring and performance insights.

Capability	        Description	                                                                                Benefit
Service Discovery	Automatically identifies the IP addresses and ports where services are exposed.	            Simplifies inter-service communication without manual setup.
Health Checks	    Continuously monitors the status of services and maintains a pool of healthy instances. 	Improves resilience and fault tolerance.
Load Balancing	    Routes traffic intelligently toward healthy instances, isolating or bypassing failing ones.	Optimizes resource usage and minimizes downtime.

`Installation & Setup:` Istioctl, cluster setup, Kiali visualization.
`Traffic Management:` Gateways, Virtual Services, Destination Rules.
`Resilience Features:` Fault injection, retries, circuit breaking, AB testing.
`Security:` Authentication, authorization, certificate management.
`Observability Tools:` Prometheus, Grafana, Jaeger, Kiali.

## Istio 
Istio is an open-source service mesh that simplifies
- Securing, connecting, and monitoring services
- Works with both Kubernetes and traditional workloads
- Offers traffic management, telemetry, and security

### Istio Architecture
- `Envoy Proxy:`
  1. High-performance proxy used in Istio.
  2. Deployed as a sidecar to each service (pod).
  3. Handles load balancing, security, and observability.
- `Data Plane:`
  1. Composed of Envoy proxies.
  2. Manages service-to-service communication.
- `Control Plane:`
  1. Configures proxies, enforces policies, and collects telemetry.
  2. Originally had three components:
     - Citadel: Certificate management.
     - Pilot: Service discovery and routing.
     - Galley: Configuration validation.
  3. Now unified into Istiod, simplifying management.
- `Istio Agent:`
  1. Runs in each pod alongside Envoy.
  2. Delivers configuration and secrets to the proxy


### Istio Installation & Setup 
How to install Istio on your Kubernetes cluster. There are three primary approaches:
- Using the Istioctl command-line utility
- Deploying via an Istio operator
- Installing with a Helm package
  
#### Installing istioctl
```
# curl -L https://istio.io/downloadIstio | sh -
# cd istio-1.27.3/tools/bin 
# sudo cp istioctl /usr/local/bin/
```
#### Install istio - demo profile
```
# istioctl install --set profile=demo -y
# istioctl verify-install

# kubectl get pods -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
istio-egressgateway-fc9ffdd69-qx9nv    1/1     Running   0          11m
istio-ingressgateway-59d55c975-kx46d   1/1     Running   0          11m
istiod-7c7fdc99bc-4sztw                1/1     Running   0          13m

# istioctl version
client version: 1.27.3
control plane version: 1.27.3
data plane version: 1.27.3 (2 proxies)
```

#### Deploying Our First Application on Istio
```
# kubectl label namespace default istio-injection=enabled
# kubectl apply -f /root/istio-1.27.3/samples/bookinfo/platform/kube/bookinfo.yaml

# kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-77d6bd5675-c6d6s      2/2     Running   0          23h
productpage-v1-bb87ff47b-ql9fx   2/2     Running   0          23h
ratings-v1-8589f64b4c-nhpsm      2/2     Running   0          23h
reviews-v1-8cf7b9cc5-nk8ld       2/2     Running   0          23h
reviews-v2-67d565655f-kmxj9      2/2     Running   0          23h
reviews-v3-d587fc9d7-ss7w9       2/2     Running   0          23h
```

#### Visualizing Service Mesh with Kiali
```
# kubectl apply -f /root/istio-1.27.3/samples/addons 
# kubectl port-forward svc/kiali -n istio-system 20001:20001

# kubectl get pods -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-6c689999f9-5ksq7               1/1     Running   0          22h
istio-egressgateway-fc9ffdd69-qx9nv    1/1     Running   0          23h
istio-ingressgateway-59d55c975-kx46d   1/1     Running   0          23h
istiod-7c7fdc99bc-4sztw                1/1     Running   0          23h
jaeger-555f5df568-zthcn                1/1     Running   0          22h
kiali-7fc595c8b7-knjw4                 1/1     Running   0          22h
loki-0                                 2/2     Running   0          22h
prometheus-798549cbb5-9wppm            2/2     Running   0          22h
```

Create Demo Traffic Into Your Mesh

```
# kubectl apply -f /root/istio-1.27.3/samples/bookinfo/networking/bookinfo-gateway.yaml

# kubectl get gateway bookinfo-gateway -o yaml
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1","kind":"Gateway","metadata":{"annotations":{},"name":"bookinfo-gateway","namespace":"default"},"spec":{"selector":{"istio":"ingressgateway"},"servers":[{"hosts":["*"],"port":{"name":"http","number":8080,"protocol":"HTTP"}}]}}
  creationTimestamp: "2025-10-29T18:44:18Z"
  generation: 1
  name: bookinfo-gateway
  namespace: default
  resourceVersion: "351714"
  uid: 6fb56f9c-f717-49ee-808c-a5867e396084
spec:
  selector:
    istio: ingressgateway
  servers:
  - hosts:
    - '*'
    port:
      name: http
      number: 8080
      protocol: HTTP

# kubectl get vs bookinfo -o yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1","kind":"VirtualService","metadata":{"annotations":{},"name":"bookinfo","namespace":"default"},"spec":{"gateways":["bookinfo-gateway"],"hosts":["*"],"http":[{"match":[{"uri":{"exact":"/productpage"}},{"uri":{"prefix":"/static"}},{"uri":{"exact":"/login"}},{"uri":{"exact":"/logout"}},{"uri":{"prefix":"/api/v1/products"}}],"route":[{"destination":{"host":"productpage","port":{"number":9080}}}]}]}}
  creationTimestamp: "2025-10-29T18:44:18Z"
  generation: 1
  name: bookinfo
  namespace: default
  resourceVersion: "351715"
  uid: e03cc1b3-eb76-4036-8a48-adf3b6a7e111
spec:
  gateways:
  - bookinfo-gateway
  hosts:
  - '*'
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
		  

# while sleep 0.01; do curl -s "http://$INGRESS_HOST:$INGRESS_PORT/productpage" &> /dev/null; done

```

#### Expose istio-ingressgateway as NodePort Service 

apiVersion: v1
kind: Service
metadata:
  name: istio-ingressgateway
  namespace: istio-system
spec:
  type: NodePort
  selector:
    istio: ingressgateway
  ports:
    - name: http2
      port: 80
      nodePort: 31380
      targetPort: 8080
    - name: https
      port: 443
      nodePort: 31390
      targetPort: 8443


### Envoy in Service Mesh Architecture
- What is a Proxy?
  A proxy acts as an intermediary between users and applications.
  It handles tasks like TLS encryption, authentication, and retries—offloading these from the application itself.
- Introduction to Envoy:
  Envoy is an open-source proxy developed by Lyft in 2015.
  Designed for modern microservices and distributed systems.
  Became part of the Cloud Native Computing Foundation (CNCF) in 2017 and graduated in 2018.
- Role in Service Mesh:
  Envoy is typically deployed as a sidecar container alongside application containers.
  It manages all inbound and outbound traffic for pods.
  Enhances routing, observability, and security without modifying application code.
- Best Practice:
  Using Envoy as a sidecar ensures consistent traffic management across microservices.
  It’s a core component of Istio, which relies on Envoy to manage and secure service communication.
