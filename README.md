# mlflow-demo

https://mlflow.org/docs/latest/deployment/deploy-model-to-kubernetes/tutorial.html

https://kserve.github.io/website/latest/get_started/
https://kserve.github.io/website/latest/get_started/first_isvc/

I had to uninstall the cert manager on my local cluster as kserve needs at least version 1.15

## Steps

### Installation of Kserve and prerequisites

```bash
curl -s "https://raw.githubusercontent.com/kserve/kserve/release-0.14/hack/quick_install.sh" | bash
```

Follow the tutorial at: https://kserve.github.io/website/latest/get_started/first_isvc/

Spin up a Pod for testing:
```bash
kubectl run --rm -it --image=alpine/curl:latest test-client -- /bin/sh
```

Test the endpoint:
```bash
curl -X GET http://sklearn-iris.kserve-test.svc.cluster.local

# Response
#{"status":"alive"}
```

Test the predict endpoint:
```bash
curl -v http://sklearn-iris.kserve-test.svc.cluster.local/v1/models/sklearn-iris:predict \
     -H "Content-Type: application/json" \
     -d '{
            "instances": [
                [6.8,  2.8,  4.8,  1.4],
                [6.0,  3.4,  4.5,  1.6]
            ]
        }' 

# Response
#{"predictions":[1,1]}
```