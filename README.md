This configuration allows to run Quant Overledger Network Gateway and a MongoDB in a local cluster

# Pre-requistes

helmfile https://github.com/roboll/helmfile and Helm V3

# Deploying overledger-network-gateway in a local kubernetes cluster
kind is a tool for running local Kubernetes clusters using Docker container “nodes”. To install it check https://kind.sigs.k8s.io/

```
kind create cluster --config kind/config
```

`NOTE`
Make sure you have enough CPU and Memory. At least 2GB of available RAM is needed

## Deploy quant-overledger-network-gateway
```
helmfile  -f helmfile.yaml apply --set environment.GATEWAY_ID=YOUR_KEY
```

## Enable ingress
To avoid to have to do Kubernetes port-forwards to access the emulator:

```
helmfile  -f helmfile.yaml apply --set ingress.enabled=true
```

And install an Ingress Controller configured for Kind (https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

## Public IP configuration when running in a local development environment

GATEWAY_HOST needs to be set to the public IP. ngrok (https://ngrok.com/) is a tool that allows to expose local servers
with public URLs. Install ngrok and run:

In a separate terminal run:

```
ngrok http 8080
```

You'll get the public URL e.g 
```
ngrok by @inconshreveable                                                                                                                                           (Ctrl+C to quit)

Session Status                online
Session Expires               7 hours, 59 minutes
Version                       2.3.35
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://bdc4806b9sdfd.ngrok.io -> http://localhost:8080
Forwarding                    https://bdc4806b9sdf9d.ngrok.io -> http://localhost:8080

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

Then you can run

```
helmfile  -f helmfile.yaml apply --set environment.GATEWAY_ID=YOUR_KEY --set ingress.enabled=true  --set environment.GATEWAY_HOST=http://deff7cab3935.ngrok.io --set ingress.hosts\[0\].host=deff7cab3935.ngrok.io --set ingress.hosts\[0\].paths\[0\]=/ 
```

**If you are NOT using zsh, then you may  not need to escape square brackets like \[0\].**



# Readiness and liveness
There are no documented health endpoints, so readiness and livensss endpoints are disabled


# Issues

In https://github.com/quantnetwork/overledger-network-gateway there's no documentation on how to configure the gateway
to use credentials when authenticating to Mongo nor MongoDB port