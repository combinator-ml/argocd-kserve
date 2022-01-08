# Infrastructure

## Development

There are (as usual) several installation options for KServe. The simplest is the bash one-liner from the [KServe docs](https://kserve.github.io/website/get_started/).

```
curl -s "https://raw.githubusercontent.com/kserve/kserve/release-0.7/hack/quick_install.sh" | bash
```

BUT, of course, it doesn't work all the time. It doesn't work on M1 macs. It also installs it in demo mode, which is open to the public, so might not be appropriate security-wise. It also requests a LoadBalancer for the istio-ingressgateway which again, is probably not what you want.

Another option is to faff around trying to get it all installed via custom Kustomize scripts. Painful, I know. But this is my recommended way, because it provides more control.

But even in this mode, I have configured it for demo mode, which means the limits aren't set properly and everything is open.

### Installation

```
# Create a cluster
kind create cluster # Or equivalent

kubectl apply -k infra/manifests
# Wait until everything is running, you will see CRD-related errors

kubectl apply -k infra/manifests # Run again to install remaining components
# Wait until everything is running
```

### Example KServe Usage

```
# Create test namespace
kubectl create ns kserve-test
# Apply KServe inference service
kubectl -n kserve-test apply -f https://raw.githubusercontent.com/kserve/kserve/master/docs/samples/v1beta1/sklearn/v1/sklearn.yaml
# Wait until it is running
```

```
k -n istio-system port-forward svc/istio-ingressgateway 8080:80 # or equivalent
```

Note that the service will 404 unless it is successfully running.

```
# Download test data
wget https://raw.githubusercontent.com/kserve/kserve/master/docs/samples/v1beta1/sklearn/v1/iris-input.json
# Query inference service with data
curl -H "Host: sklearn-iris.kserve-test.example.com" localhost:8080/v1/models/sklearn-iris:predict -d @./iris-input.json -v
```

It should result in `{"predictions": [1, 1]}`.


## Troubleshooting

* CRD warnings
Don't worry about it. The istio operator installs the CRDs. It will work once the istio gateways are running.

* Istio gateway doesn't start

I've observed the istio operator just not run. No idea why. So the istio stuff never gets installed. 

Delete the istiooperator pod and tail the logs. It should work now. Then continue the second apply.

* KServe pod is crashlooping

KServe won't work unless istio is correctly installed. It's likely that you've had a problem elsewhere. Delete the pod once the cluster is ready to restart.