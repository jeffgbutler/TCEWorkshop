# Contour Deep Dive

The app-toolkit is a composite package that includes many other packages. One of the packages installed is Contour - 
an ingress controller. In this section, we'll take a quick look at Contour and how to use it. This will be helpful when
we create a custom supply chain with Cartographer.

Contour is open source. The home page is here: https://projectcontour.io/

## Contour as a Kubernetes Ingress Controller

Contour listens for creation of standard Kubernetes Ingress objects and implements the routing for them.

Here is a snippet from the file [contour-test-ingress.yaml](contour-test-ingress.yaml) in this directory:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kuard-ingress
  labels:
    app: ingress
spec:
  rules:
  - host: kuard-ingress.127-0-0-1.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kuard-ingress
            port:
              number: 8080
```

This is a standard Kubernetes ingress specification. It matches any http (or https) request starting with
`kuard-ingress.127-0-0-1.nip.io` and forwards the request to a service named `kuard-ingress` on port 8080.

You can try this out by running the following command:

```shell
kubectl apply -f contour-test-ingress.yaml
```

After this runs, the Kuard application will be exposed at http://kuard-ingress.127-0-0-1.nip.io

(If you are running a cluster with a different DNS you will need to make the appropriate modifications to the
YAML before applying)

## Native Contour

Contour can also be configured with its native CRD called HTTPProxy. This method of configuring Contour attempts
to circumvent some limitations of the native Ingress controller in Kubernetes. We will not be concerned with
any of those limitation in this workshop. You can read all about the reasoning behind this
here: https://projectcontour.io/docs/main/config/fundamentals/

Here is a snippet from [contour-test-httpproxy.yaml](contour-test-httpproxy.yaml) in this directory:

```yaml
apiVersion: projectcontour.io/v1
kind: HTTPProxy
metadata:
  name: kuard-httpproxy
  labels:
    app: ingress
spec:
  virtualhost:
    fqdn: kuard-httpproxy.127-0-0-1.nip.io
  routes:
  - conditions:
    - prefix: /
    services:
    - name: kuard-httpproxy
      port: 8080
```

This code snippet is functionally equivalent to the Ingress specification above. It matches any http (or https)
request starting with `kuard-httpproxy.127-0-0-1.nip.io` and forwards the request to a service named `kuard-httpproxy`
on port 8080.

After this runs, the Kuard application will be exposed at http://kuard-httpproxy.127-0-0-1.nip.io

(If you are running a cluster with a different DNS you will need to make the appropriate modifications to the
YAML before applying)

[Next (Explore Knative) -&gt;](../04-knative/README.md)
