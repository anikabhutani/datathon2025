# SmartClinical - Risk-Driven Hospital Resource Allocation System

🏥 A web-based clinical decision-support tool that predicts patient risk levels from vital signs and automatically recommends optimized allocation of hospital resources.

## Overview

SmartClinical uses machine learning to analyze patient vital signs and predict risk levels (Normal/Low/Medium/High). Based on these predictions, the system automatically allocates limited hospital resources (oxygen units, clinical staff) to patients who need them most.

**⚠️ Important**: This is an advisory tool for resource planning, not a diagnostic or clinical decision-making system.

## Features

- **Risk Prediction Engine**: ML-powered risk assessment using patient vitals
- **Resource Allocation Algorithm**: Automated allocation of oxygen and staff based on risk severity
- **Batch Processing**: Handle multiple patients simultaneously
- **Interactive Dashboard**: Streamlit-based UI with CSV upload and manual entry
- **Export Reports**: Download allocation results as CSV or JSON
- **Real-time Visualization**: Charts and graphs for risk distribution and resource allocation

## Architecture

- **Backend**: FastAPI with RESTful API endpoints
- **Frontend**: Streamlit web application
- **ML Model**: RandomForest classifier trained on patient data
- **Data Format**: CSV with patient vital signs

## Quick Start

### Option 1: Automated Startup (Recommended)

```bash
# Install dependencies first
pip install -r requirements.txt

# Run the startup script (trains model if needed, then starts both servers)
./start.sh
```

### Option 2: Manual Startup

1. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Train the model** (if not already trained):
   ```bash
   python train_model.py
   ```
   This will create `risk_model.joblib` file.

3. **Start the API server** (Terminal 1):
   ```bash
   python api.py
   ```

4. **Start the frontend** (Terminal 2):
   ```bash
   streamlit run app.py
   ```

## Usage

### Step 1: Start the API Server

In one terminal:
```bash
python api.py
```

The API will run on `http://localhost:8000`

You can verify it's running by visiting:
- `http://localhost:8000` - API root
- `http://localhost:8000/docs` - Interactive API documentation

### Step 2: Start the Frontend

In another terminal:
```bash
streamlit run app.py
```

The dashboard will open in your browser at `http://localhost:8501`

### Step 3: Use the Application

1. **Upload Patient Data**:
   - Upload a CSV file with patient vitals, OR
   - Enter patient data manually

2. **Set Available Resources**:
   - Specify number of oxygen units available
   - Specify number of staff members available

3. **Run Allocation**:
   - Click "Run Allocation" button
   - Review the results

4. **Export Results**:
   - Download allocation report as CSV or JSON

## Data Format

### CSV Input Format

Your CSV file should have the following columns:

- `Patient_ID` (optional): Patient identifier
- `Respiratory_Rate`: Respiratory rate per minute (0-60)
- `Oxygen_Saturation`: Oxygen saturation percentage (0-100)
- `O2_Scale`: O2 scale (1, 2, or 3)
- `Systolic_BP`: Systolic blood pressure (0-300)
- `Heart_Rate`: Heart rate per minute (0-300)
- `Temperature`: Body temperature in Celsius (30-45)
- `Consciousness`: Consciousness level - A (Alert), P (Pain), U (Unresponsive), V (Verbal)
- `On_Oxygen`: On oxygen - 0 (No) or 1 (Yes)

Example:
```csv
Patient_ID,Respiratory_Rate,Oxygen_Saturation,O2_Scale,Systolic_BP,Heart_Rate,Temperature,Consciousness,On_Oxygen
P0001,25,96,1,97,107,37.5,A,0
P0002,28,92,2,116,151,38.5,P,1
```

## API Endpoints

### `GET /`
Root endpoint with API information

### `GET /health`
Health check endpoint

### `POST /predict`
Predict risk for a single patient

**Request**:
```json
{
  "patient": {
    "Respiratory_Rate": 25,
    "Oxygen_Saturation": 96,
    "O2_Scale": 1,
    "Systolic_BP": 97,
    "Heart_Rate": 107,
    "Temperature": 37.5,
    "Consciousness": "A",
    "On_Oxygen": 0
  }
}
```

**Response**:
```json
{
  "risk_level": "Medium",
  "risk_score": 0.456,
  "probabilities": {
    "Normal": 0.123,
    "Low": 0.234,
    "Medium": 0.456,
    "High": 0.187
  }
}
```

### `POST /allocate`
Allocate resources for a batch of patients

**Request**:
```json
{
  "patients": [
    {
      "patient_id": "P0001",
      "Respiratory_Rate": 25,
      "Oxygen_Saturation": 96,
      "O2_Scale": 1,
      "Systolic_BP": 97,
      "Heart_Rate": 107,
      "Temperature": 37.5,
      "Consciousness": "A",
      "On_Oxygen": 0
    }
  ],
  "available_oxygen": 10,
  "available_staff": 5
}
```

**Response**:
```json
{
  "allocation": [
    {
      "patient_id": "P0001",
      "risk_level": "Medium",
      "risk_score": 0.456,
      "probabilities": {...},
      "allocated_oxygen": true,
      "allocated_staff": false,
      "vitals": {...}
    }
  ],
  "summary": {
    "total_patients": 1,
    "high_risk": 0,
    "medium_risk": 1,
    "low_risk": 0,
    "normal": 0,
    "oxygen_allocated": 1,
    "staff_allocated": 0,
    "available_oxygen": 10,
    "available_staff": 5
  },
  "timestamp": "2025-11-XX..."
}
```

## Allocation Logic

The system allocates resources based on the following priority:

1. **Oxygen Allocation**:
   - High-risk patients get priority
   - Medium-risk patients get oxygen if available
   - Low/Normal risk patients do not receive oxygen

2. **Staff Allocation**:
   - High-risk patients get staff assistance first
   - Other patients receive staff only if available after all High-risk patients are covered

3. **Sorting**:
   - Patients are sorted by risk level (High > Medium > Low > Normal)
   - Within the same risk level, sorted by risk score (probability)

## Model Details

- **Algorithm**: RandomForest Classifier
- **Features**: 7 numeric features + 1 categorical feature (Consciousness)
- **Classes**: Normal, Low, Medium, High
- **Training**: 80/20 train/validation split with stratification
- **Performance**: See training output for classification report

## File Structure

```
.
├── api.py                      # FastAPI backend server
├── app.py                      # Streamlit frontend
├── train_model.py              # Model training script
├── risk_model.joblib           # Trained model (generated)
├── Health_Risk_Dataset.csv     # Training dataset
├── requirements.txt            # Python dependencies
├── README.md                   # This file
└── datathonhealthrisk2025.py   # Original code (reference)
```

## Development

### Training a New Model

```bash
python train_model.py
```

### Running Tests

You can test the API endpoints using the interactive docs at `http://localhost:8000/docs` or using curl:

```bash
# Health check
curl http://localhost:8000/health

# Single prediction
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{"patient": {"Respiratory_Rate": 25, "Oxygen_Saturation": 96, "O2_Scale": 1, "Systolic_BP": 97, "Heart_Rate": 107, "Temperature": 37.5, "Consciousness": "A", "On_Oxygen": 0}}'
```

## Future Enhancements

- Model explainability (SHAP values)
- Scenario simulator for stress testing
- Audit logging for compliance
- Confidence intervals / uncertainty quantification
- EHR system integration (FHIR)
- ICU bed and ventilator allocation
- Auto-retraining pipeline

## License

This project is for educational/demonstration purposes.

## Disclaimer

**This system is advisory only and is not a diagnostic tool.** It is designed for workflow support and resource planning. All clinical decisions should be made by qualified healthcare professionals based on comprehensive patient assessment.

## Support

For issues or questions, please refer to the API documentation at `http://localhost:8000/docs` when the server is running.

