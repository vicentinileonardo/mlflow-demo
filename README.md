# mlflow Demo

References:
- https://kserve.github.io/website/latest/get_started/
- https://kserve.github.io/website/latest/get_started/first_isvc/
- https://mlflow.org/docs/latest/deployment/deploy-model-to-kubernetes/tutorial.html

Notes:

I had to uninstall cert-manager on my local cluster as KServe needs at least version 1.15.
Therefore cert-manager is installed by the KServe installation script.

This demo is not leveraging the MLflow model registry.

## Steps

### Installation of KServe and prerequisites on the k8s cluster

```bash
curl -s "https://raw.githubusercontent.com/kserve/kserve/release-0.14/hack/quick_install.sh" | bash
```

### Model preparation

#### Training the model
```bash
python model.py
```

#### Testing the model locally
Not done (due to local environment issues)

#### Building the Docker image of model
Done inside the GitHub Actions workflow

#### Pushing the Docker image to DockerHub
Done inside the GitHub Actions workflow

### Model deployment

#### Creating K8s namespace
```bash
kubectl create namespace mlflow-kserve-test
```

#### Deploying the KServe InferenceService referencing the Docker image of the model
```bash
kubectl apply -f deployment.yaml
```

#### Checking the status of the InferenceService
```bash
kubectl get inferenceservice mlflow-wine-classifier -n mlflow-kserve-test
```


### Testing the InferenceService

#### Spinning up a test client
```bash
kubectl run --rm -it --image=alpine/curl:latest test-client -- /bin/sh
```

#### Testing health endpoints
```bash
# Check if the server is ready
curl -v -X GET http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/health/ready
# Response
# 200 OK

# Check if the server is live
curl -v -X GET http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/health/live
# Response
# 200 OK

# GET server metadata
curl -v -X GET http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/
# Response
# {"name":"mlserver","version":"1.3.5","extensions":[]}
```

#### Getting available models
```bash
curl -v http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/repository/index \
     -H "Content-Type: application/json" \
     -d '{}' 
# Response
# [{"name":"mlflow-model","version":"dce3966053da44658dc78889d02ac9d8","state":"READY","reason":""}]
```

#### Getting model metadata
```bash
curl -v -X GET http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/models/mlflow-model
# Response
# {"name":"mlflow-model","versions":[],"platform":"","inputs":[{"name":"input-0","datatype":"FP64","shape":[-1,13],"parameters":{"content_type":"np"}}],"outputs":[{"name":"output-0","datatype":"FP64","shape":[-1],"parameters":{"content_type":"np"}}],"parameters":{"content_type":"np"}}
```

#### Checking if the model is ready
```bash
curl -v -X GET http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/models/mlflow-model/ready
# Response
# 200 OK
```

#### Testing the infer endpoint
```bash
curl -v http://mlflow-wine-classifier.mlflow-kserve-test.svc.cluster.local/v2/models/mlflow-model/infer \
     -H "Content-Type: application/json" \
     -d '{
    "inputs": [
        {
            "name": "input",
            "shape": [1, 13],
            "datatype": "FP64",
            "data": [[14.23, 1.71, 2.43, 15.6, 127.0, 2.8, 3.06, 0.28, 2.29, 5.64, 1.04, 3.92, 1065.0]]
        }
    ]
}' 

# Response
# {"model_name":"mlflow-model","model_version":"dce3966053da44658dc78889d02ac9d8","id":"************************************","parameters":{"content_type":"np"},"outputs":[{"name":"output-1","shape":[1,1],"datatype":"FP64","parameters":{"content_type":"np"},"data":[0.44111927902918846]}]}
```

##### Input data explanation

MLflow automatically infers the schema of the input data and the output data.
The model expects 13 features as input. We can check shape of the input data and feature names as follows in the Python code of the model:

```python
X, y = datasets.load_wine(return_X_y=True)
print(X.shape, y.shape)
print(datasets.load_wine().feature_names)

# Output
# (178, 13) (178,)
# ['alcohol', 'malic_acid', 'ash', 'alcalinity_of_ash', 'magnesium', 'total_phenols', 'flavanoids', 'nonflavanoid_phenols', 'proanthocyanins', 'color_intensity', 'hue', 'od280/od315_of_diluted_wines', 'proline']
```
