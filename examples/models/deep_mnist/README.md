# Tensorflow MNIST
A TensorFlow model for MNIST classification.

## Dependencies

```bash
pip install tensorflow
```

## Train locally

```bash
make train
```

## Wrap using [s2i](https://github.com/openshift/source-to-image#installation).

```bash
s2i build . seldonio/seldon-core-s2i-python2 deep-mnist
```

## Local Docker Smoke Test

Run under docker.

```bash
docker run --rm -p 5000:5000 deep-mnist
```

Ensure test grpc modules compiled.

```bash
pushd ../../../wrappers/testing ; make build_protos ; popd
```

Send a data request using the wrapper tester.

```bash
python ../../../wrappers/testing/tester.py contract.json 0.0.0.0 5000 -p
```

## Minikube test

```bash
minikube start --memory 4096
```

Install seldon core

```
helm install seldon-core-crd --name seldon-core-crd --repo https://storage.googleapis.com/seldon-charts --set usage_metrics.enabled=true --set rbac.enabled=false
helm install seldon-core --name seldon-core --repo https://storage.googleapis.com/seldon-charts --set rbac.enabled=false
```

Connect to Minikube Docker daemon

```bash
eval $(minikube docker-env)
```

Build image using minikube docker daemon.

```bash
s2i build . seldonio/seldon-core-s2i-python2 deep-mnist
```

Launch deployment

```bash
kubectl create -f deep_mnist.json
```

Port forward API server

```bash
kubectl port-forward $(kubectl get pods -l app=seldon-apiserver-container-app -o jsonpath='{.items[0].metadata.name}') 8080:8080
```

Ensure tester gRPC modules compiled. You will need to install grpc-tools module.

```bash
pushd ../../../util/api_tester ; make build_protos ; popd
```

Send test request
```bash
python ../../../util/api_tester/api-tester.py contract.json 0.0.0.0 8080 --oauth-key oauth-key --oauth-secret oauth-secret -p
```


