# Healthcare Billing Prediction

A machine learning project that trains multiple regression models to **predict hospital billing amounts** based on patient and admission data, and stores predictions in **MongoDB**.

---

## Project Structure

```text
temp/
├── healthcare_dataset.csv   # Dataset downloaded from Kaggle
├── train_model.py           # Train & compare regression models
├── mongodb_handler.py       # MongoDB CRUD + analytics operations
├── billing_model.pkl        # Saved best model (generated after training)
├── requirements.txt         # Python dependencies
└── venv/                    # Python virtual environment
```

---

## Dataset

**Source:** [Healthcare Dataset on Kaggle](https://www.kaggle.com/datasets/prasad22/healthcare-dataset)

Download `healthcare_dataset.csv` and place it in the project folder before running the project.

**Target variable:** `Billing Amount`

**Input features used:**

| Feature | Type |
|---|---|
| Age | Numeric |
| Gender | Categorical |
| Blood Type | Categorical |
| Medical Condition | Categorical |
| Insurance Provider | Categorical |
| Admission Type | Categorical |
| Medication | Categorical |
| Test Results | Categorical |
| Length of Stay (engineered) | Numeric |
| Admission Month (engineered) | Numeric |

---

## Setup (Windows)

### 1. Install Python 3.11

Download Python 3.11 from:

https://www.python.org/downloads/

During installation, select **Add Python to PATH**.

Verify installation in PowerShell:

```powershell
py -3.11 --version
```

### 2. Open the project folder

Open PowerShell and navigate to your project directory.

Example:

```powershell
cd "$HOME\Downloads\mongo2\temp"
```

Replace the path if your project is stored somewhere else.

### 3. Create and activate the virtual environment

Create a virtual environment using Python 3.11:

```powershell
py -3.11 -m venv venv
```

Activate it:

```powershell
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks script execution, run:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Then activate again:

```powershell
.\venv\Scripts\Activate.ps1
```

You should now see `(venv)` at the beginning of your terminal prompt.

**Important:** Activate the virtual environment every time you open a new terminal.

### 4. Upgrade pip

```powershell
python -m pip install --upgrade pip
```

### 5. Install dependencies

```powershell
python -m pip install -r requirements.txt
```

### 6. Place the dataset

Download `healthcare_dataset.csv` from Kaggle.

Move it from Downloads into your project folder:

```powershell
Move-Item "$HOME\Downloads\healthcare_dataset.csv" .
```

Verify that the file exists:

```powershell
dir
```

---

## Usage

**Always activate your virtual environment before running the scripts.**

```powershell
cd "$HOME\Downloads\mongo2\temp"
.\venv\Scripts\Activate.ps1
```

### Step 1 — Train the model

```powershell
python train_model.py
```

This will:

- Load and clean `healthcare_dataset.csv`.
- Engineer `LengthOfStay` and `AdmissionMonth` features.
- Train 5 regression models.
- Compare model performance using MAE, RMSE and R².
- Select the best model based on the lowest MAE.
- Save the complete best-performing model pipeline as `billing_model.pkl`.

**Models compared:**

| Model | Configuration |
|---|---|
| Dummy Regressor | Baseline (predicts mean) |
| Linear Regression | Simple linear model |
| Decision Tree | `min_samples_leaf=10` |
| Random Forest | 100 trees, `min_samples_leaf=5` |
| Gradient Boosting | `learning_rate=0.05`, `max_depth=2` |

### Step 2 — Start MongoDB

Make sure MongoDB Community Server is installed on Windows.

Open PowerShell as Administrator and start the MongoDB service:

```powershell
Start-Service MongoDB
```

Check its status:

```powershell
Get-Service MongoDB
```

The service should show `Running`.

If MongoDB was installed without a Windows service, start the server manually using its installed `mongod.exe` executable and configured data directory.

### Step 3 — Run the MongoDB handler

Open another PowerShell terminal, navigate to the project folder, and activate the virtual environment:

```powershell
cd "$HOME\Downloads\mongo2\temp"
.\venv\Scripts\Activate.ps1
```

Run:

```powershell
python mongodb_handler.py
```

This opens an interactive menu with 15 options:

```text
================================================
  Healthcare Billing — MongoDB Handler
================================================
  1. Predict billing & save to MongoDB
  2. Show recent records (last 10)
  3. Find record by ID
  4. Filter records by billing threshold
  5. Filter records by medical condition
  6. Filter records by patient age
  7. Sort records by predicted billing
  8. Add / update note on a record
  9. Correct predicted billing value
 10. Delete one record
 11. Delete ALL records
 12. Count records
 13. Aggregation summary by condition
 14. Aggregation summary by insurance
 15. Aggregation summary by admission type
  0. Exit
================================================
```

**Note:** Option 1 requires `billing_model.pkl` to exist. Always run `train_model.py` before using the prediction option.

---

## MongoDB Configuration

- **Database:** `healthcare_db`
- **Collection:** `prediction_history`
- **Connection:** `mongodb://localhost:27017/`

Make sure MongoDB is running before launching the handler.

### Record Schema

Each saved prediction contains:

```json
{
  "name": "Alice",
  "age": 45,
  "gender": "Female",
  "blood_type": "A+",
  "medical_condition": "Diabetes",
  "insurance_provider": "Aetna",
  "admission_type": "Elective",
  "medication": "Metformin",
  "test_results": "Normal",
  "length_of_stay": 7,
  "admission_month": 3,
  "predicted_billing": 18432.50,
  "timestamp": "2026-11-03T12:00:00"
}
```

---

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| `pandas` | 3.1.0 | Data loading and manipulation |
| `numpy` | 2.4.6 | Numerical operations |
| `scikit-learn` | 1.9.1 | ML models and preprocessing |
| `joblib` | 1.6.0 | Model serialization |
| `pymongo` | 4.19.0 | MongoDB driver (`bson` included) |

Install all dependencies:

```powershell
python -m pip install -r requirements.txt
```

---

## Important Notes

- **Python 3.11:** Recommended environment for this project. Use compatible package versions.
- **Virtual environment:** The `venv/` folder is local and should be included in `.gitignore`.
- **Model regeneration:** `billing_model.pkl` is regenerated whenever `train_model.py` runs. The selected model may change depending on the dataset and evaluation results.
- **MongoDB:** Ensure the MongoDB server is running before starting `mongodb_handler.py`.
- **Prediction timing:** Since the model uses length of stay and test results, it predicts billing at discharge, not before admission.
- **Negative billing values:** The training script excludes negative billing amounts under the assumption that they are invalid rather than legitimate refunds or credits.

---

## .gitignore

Add the following entries to your `.gitignore` file:

```text
venv/
__pycache__/
*.pyc
billing_model.pkl
```

You can also exclude the dataset if you do not want to upload it to GitHub:

```text
healthcare_dataset.csv
```

---

## Project Execution Summary

Run these commands in PowerShell after installing the dependencies and placing the dataset:

```powershell
# Navigate to project
cd "$HOME\Downloads\mongo2\temp"

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Train regression models
python train_model.py

# Start MongoDB (run separately in an Administrator terminal)
Start-Service MongoDB

# Run MongoDB handler
python mongodb_handler.py
```

**Project workflow:**

Dataset → Data Preprocessing → Feature Engineering → Multiple Regression Models → Model Evaluation → Best Model Saved → Billing Prediction → MongoDB Storage → CRUD Operations & Analytics
# temp
