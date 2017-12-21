# envoymesh

Envoy mesh is an experimental simple service mesh built on top of
[go-control-plane](https://github.com/envoyproxy/go-control-plane) that
provides the following features:

- sidecar-based service mesh architecture
- lightweight installation, targeted for Kubernetes
- out-of-the-box telemetry, authorization checks, and L7 routing capabilities
- direct access to [Envoy xDS
  APIs](https://github.com/envoyproxy/data-plane-api) for customizing
  application-level network behavior

## Goals
- Minimal implementation of a control plane for a fleet of Envoy proxies
- ADS for coordinated configuration rollout
- Implementation of native Envoy extension point (access log, metrics, authz)

## Limitations

- This project uses jsonnet extensively for rapid prototyping of Envoy API
  processing logic.
- To simplify the deployment model, sidecar container includes both the proxy
  and its controller.
- No support for health checks in the application deployment.

## Build instructions

envoymesh uses standard go tooling. Requirements:
- golang 1.9.2 or above
- godep

Use `build.sh` script to generate a sidecar container that includes
[envoy](https://www.envoyproxy.io/) and a controller binary.

## Test instructions

1. Use the famous bookinfo app for demonstration:

        kubectl apply -f samples/bookinfo.yaml

Access the web page by using `EXTERNAL_IP` of `productpage` service:
`http://EXTERNAL_IP/productpage`

2. Grant admin permissions to the application service account:
    
        kubectl create clusterrolebinding envoymesh --clusterrole=cluster-admin --serviceaccount=default:envoymesh

3. Inject the sidecar using the following script:

        cat samples/bookinfo.yaml \
          | go run cmd/inject/main.go \
          > samples/bookinfo-injected.yaml 

        kubectl apply -f samples/bookinfo-injected.yaml

Access the web page again at `http://EXTERNAL_IP/productpage`. Traffic should
be flowing through Envoy!

