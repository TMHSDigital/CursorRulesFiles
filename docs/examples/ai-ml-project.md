# AI/ML Project Example

This example demonstrates how to use CursorRules in an AI/ML project with FastAPI backend and React frontend.

## Project Setup

1. Create project directory structure:
```
/my-ai-project
  /.cursorrules              # Directory for rule files
    project.cursorrules      # Core project rules
    ai_guidelines.cursorrules # AI-specific rules
    modern_practices.cursorrules # Modern AI practices
  /src
    /backend                 # FastAPI backend
      /api
        /v1
          /endpoints
          /models
      /core
        /ml
          /models          # ML models
          /training       # Training scripts
          /inference     # Inference logic
          /evaluation    # Model evaluation
        /data
          /processing    # Data processing
          /validation    # Data validation
          /storage      # Data storage
    /frontend              # React frontend
      /src
        /components
        /pages
        /hooks
    /notebooks             # Jupyter notebooks
    /tests
      /unit
      /integration
      /model_tests
  /data                    # Data files
    /raw                  # Raw data
    /processed           # Processed data
    /models              # Saved models
  requirements.txt
  pyproject.toml
```

2. Copy relevant rules:
```bash
# Copy all needed rules
cp rules/core/project.cursorrules .cursorrules/
cp rules/ai/ai_guidelines.cursorrules .cursorrules/
cp rules/ai/modern_practices.cursorrules .cursorrules/
```

## Rule Customization

### Project Rules
```
// project.cursorrules
// AI/ML project setup

// Technology Stack
- ML Framework: PyTorch/TensorFlow
- API: FastAPI
- Frontend: React + Vite
- Data Processing: Pandas, NumPy
- Visualization: Plotly
- Experiment Tracking: MLflow
- Model Registry: MLflow

// Project Structure
/src
  /backend             # FastAPI backend
    /api             # API endpoints
    /core           # Core ML logic
      /ml          # ML components
      /data       # Data handling
  /frontend          # React frontend
  /notebooks         # Jupyter notebooks
/data                # Data directory
  /raw              # Raw data
  /processed       # Processed data
  /models          # Saved models
```

### AI Guidelines
```
// ai_guidelines.cursorrules
// AI/ML specific patterns

// Model Development
- Training Pipeline:
  * Data preprocessing
  * Model architecture
  * Training loop
  * Validation
  * Model saving

// Example Training Script
import torch
from torch import nn
from torch.utils.data import DataLoader
import mlflow

class ModelTrainer:
    def __init__(self, model: nn.Module, config: dict):
        self.model = model
        self.config = config
        self.device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
        
        mlflow.set_experiment(config['experiment_name'])
        
    def train(self, train_loader: DataLoader, val_loader: DataLoader):
        with mlflow.start_run():
            # Log parameters
            mlflow.log_params(self.config)
            
            # Training loop
            for epoch in range(self.config['epochs']):
                train_loss = self._train_epoch(train_loader)
                val_loss = self._validate(val_loader)
                
                # Log metrics
                mlflow.log_metrics({
                    'train_loss': train_loss,
                    'val_loss': val_loss
                }, step=epoch)
            
            # Save model
            mlflow.pytorch.log_model(self.model, 'model')
    
    def _train_epoch(self, loader: DataLoader) -> float:
        self.model.train()
        total_loss = 0
        
        for batch in loader:
            loss = self._process_batch(batch)
            total_loss += loss.item()
            
        return total_loss / len(loader)
```

### Modern Practices
```
// modern_practices.cursorrules
// Modern AI development practices

// Model Versioning
- Version Control:
  * Use DVC for data versioning
  * Track experiments with MLflow
  * Version models in registry
  * Document hyperparameters
  * Track metrics

// Inference API
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import torch

app = FastAPI()

class PredictionRequest(BaseModel):
    features: list[float]

class PredictionResponse(BaseModel):
    prediction: float
    confidence: float

@app.post('/predict', response_model=PredictionResponse)
async def predict(request: PredictionRequest):
    try:
        # Preprocess
        features = torch.tensor(request.features)
        
        # Inference
        with torch.no_grad():
            prediction = model(features)
            confidence = calculate_confidence(prediction)
        
        return PredictionResponse(
            prediction=prediction.item(),
            confidence=confidence
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

## Implementation Examples

### Data Processing
```python
# src/core/data/processing.py
import pandas as pd
from sklearn.preprocessing import StandardScaler
from typing import Tuple

class DataProcessor:
    def __init__(self, config: dict):
        self.config = config
        self.scaler = StandardScaler()
    
    def process_data(
        self,
        data: pd.DataFrame
    ) -> Tuple[pd.DataFrame, pd.Series]:
        # Validation
        self._validate_data(data)
        
        # Feature engineering
        features = self._engineer_features(data)
        
        # Split features and target
        X = features.drop(self.config['target_column'], axis=1)
        y = features[self.config['target_column']]
        
        # Scaling
        X_scaled = pd.DataFrame(
            self.scaler.fit_transform(X),
            columns=X.columns
        )
        
        return X_scaled, y
```

### Model Architecture
```python
# src/core/ml/models/model.py
import torch
import torch.nn as nn

class MLPModel(nn.Module):
    def __init__(
        self,
        input_dim: int,
        hidden_dims: list[int],
        output_dim: int
    ):
        super().__init__()
        
        layers = []
        prev_dim = input_dim
        
        for hidden_dim in hidden_dims:
            layers.extend([
                nn.Linear(prev_dim, hidden_dim),
                nn.ReLU(),
                nn.BatchNorm1d(hidden_dim),
                nn.Dropout(0.2)
            ])
            prev_dim = hidden_dim
        
        layers.append(nn.Linear(prev_dim, output_dim))
        self.network = nn.Sequential(*layers)
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.network(x)
```

### Model Evaluation
```python
# src/core/ml/evaluation/metrics.py
import numpy as np
from sklearn.metrics import (
    accuracy_score,
    precision_recall_fscore_support
)
from typing import Dict

def evaluate_model(
    y_true: np.ndarray,
    y_pred: np.ndarray
) -> Dict[str, float]:
    accuracy = accuracy_score(y_true, y_pred)
    precision, recall, f1, _ = precision_recall_fscore_support(
        y_true,
        y_pred,
        average='weighted'
    )
    
    return {
        'accuracy': accuracy,
        'precision': precision,
        'recall': recall,
        'f1': f1
    }
```

## Best Practices

1. Model Development:
   - Version data and models
   - Track experiments
   - Document hyperparameters
   - Validate results

2. Code Quality:
   - Type hints
   - Documentation
   - Unit tests
   - Integration tests

3. Deployment:
   - Model serving
   - API endpoints
   - Monitoring
   - Version control

4. Data Management:
   - Data versioning
   - Data validation
   - Data privacy
   - Data documentation 