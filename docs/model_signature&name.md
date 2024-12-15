The input shape in MLflow is typically set during the model training and saving process. There are several ways this can be defined:

Using scikit-learn's Pipeline or Direct Model Export:
When you save a model using MLflow, the input schema can be captured through:
- Automatic schema inference during model training
- Explicit schema definition using mlflow.pyfunc.log_model() or mlflow.sklearn.log_model()

Example of setting schema explicitly during model saving:
```python
import mlflow
import mlflow.sklearn
import numpy as np
from sklearn.pipeline import Pipeline

# Create your model/pipeline
model = ...  # Your trained model

# Log the model with explicit signature
signature = mlflow.models.ModelSignature(
    inputs=mlflow.types.Schema([
        mlflow.types.ColSpec(type_constraint="double", name="input", shape=(-1, 13))
    ]),
    outputs=mlflow.types.Schema([
        mlflow.types.ColSpec(type="double")
    ])
)

mlflow.sklearn.log_model(
    model, 
    "model", 
    signature=signature
)
```

In the MLmodel file, the schema is typically represented like:
```yaml
signature:
  inputs: '[{"type": "tensor", "tensor-spec": {"dtype": "float64", "shape": [-1, 13]}}]'
  outputs: '[{"type": "tensor", "tensor-spec": {"dtype": "float64", "shape": [-1, 1]}}]'
```

---

In MLflow, you can set the model name when logging a model in a few different ways:

Using mlflow.log_model():

```python
mlflow.log_model(
    model, 
    artifact_path="model",
    registered_model_name="your_model_name"
)
```

If you're using scikit-learn, PyTorch, or other framework-specific logging methods:

```python
mlflow.sklearn.log_model(
    model, 
    artifact_path="model",
    registered_model_name="your_model_name"
)
```

When saving a model in a run:
```python
mlflow.start_run():
    mlflow.log_model(
        model, 
        artifact_path="model",
        registered_model_name="your_model_name"
    )
```


