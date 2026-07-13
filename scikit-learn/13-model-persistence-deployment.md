# 13 — Model Persistence & Deployment Basics

Training a model is only useful if it can actually be SAVED and reused later, without retraining from scratch every single time. This file covers the practical "now what do I do with a trained model" step, which genuinely doesn't get enough attention in a lot of tutorials that stop right after `.fit()`.

## joblib — the standard way to save scikit-learn models
```python
import joblib

joblib.dump(model, "my_model.pkl")     # save a trained model to disk
loaded_model = joblib.load("my_model.pkl")     # load it back later, fully trained and ready
predictions = loaded_model.predict(X_new)         # use it immediately, no retraining needed
```
`joblib` is generally preferred over Python's built-in `pickle` for scikit-learn models specifically, since it's more efficient at handling the large NumPy arrays that trained models typically contain internally (like a Random Forest's many trees, or a linear model's weight array).

## Saving an entire Pipeline (recap from file 10 — genuinely the realistic way to do this)
```python
joblib.dump(full_pipeline, "complete_pipeline.pkl")     # saves preprocessing steps AND the model together
loaded_pipeline = joblib.load("complete_pipeline.pkl")
predictions = loaded_pipeline.predict(X_new_raw)             # can predict directly on RAW new data -- preprocessing happens automatically
```
This is genuinely the version I'd actually use in practice — saving just a bare model without its accompanying preprocessing steps means you'd have to remember and correctly reapply the exact same scaling/encoding manually every time, which is fragile and error-prone.

## Using pickle directly (the more general Python alternative)
```python
import pickle

with open("model.pkl", "wb") as f:     # 'wb' = write binary
    pickle.dump(model, f)

with open("model.pkl", "rb") as f:        # 'rb' = read binary
    loaded_model = pickle.load(f)
```
Works fine too, genuinely more general-purpose (works for any Python object, not just ML models), but `joblib` is still the more common convention specifically within the scikit-learn ecosystem.

## A genuinely important caution — version compatibility
```python
import sklearn
print(sklearn.__version__)     # worth checking/recording this alongside a saved model
```
A model saved with one version of scikit-learn isn't always guaranteed to load correctly with a different version later — genuinely worth noting the exact library version used when saving a model for long-term/production use, especially since the model itself doesn't carry that information automatically.

## A minimal deployment pattern — wrapping a model in a simple API
```python
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)
model = joblib.load("complete_pipeline.pkl")     # load ONCE when the app starts, not on every request

@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json()
    X_new = pd.DataFrame([data])     # convert incoming JSON into the same DataFrame shape the model expects
    prediction = model.predict(X_new)
    return jsonify({"prediction": prediction.tolist()})
```
Haven't actually deployed anything yet, but this is the general SHAPE of how a trained model becomes something a real application can actually use — load the model once, expose an endpoint that accepts new data and returns predictions. Genuinely just noting this here as a conceptual anchor for later, not something I've built end-to-end myself.

## Checking model input requirements before deployment (a genuinely practical gotcha)
```python
# The exact SAME columns, in the SAME order, with the SAME preprocessing, must be provided at prediction time
# A common real mistake: training with columns ["age", "salary", "department"] but later feeding
# prediction data with columns in a DIFFERENT order or with a MISSING column entirely
```
This is worth remembering as a genuinely common real-world deployment bug — a model doesn't "know" column names the way a human reading a DataFrame does; if the input columns for prediction don't exactly match what was used during training (same names, same order, same encoding), predictions can silently be wrong instead of throwing an obvious error, which is genuinely worse.

## Retraining considerations (a forward-looking note, not deeply explored yet)
Real deployed models eventually need RETRAINING as new data comes in and the real-world patterns the model learned from potentially shift over time ("model drift" or "data drift") — a model trained once and deployed forever isn't realistic for most genuinely long-lived applications. Noting this exists as a real consideration, though the actual practices around monitoring/retraining schedules are more of an MLOps topic than something scikit-learn itself handles.

## Honest summary of where I am with this topic
This file is genuinely more "awareness of what exists" than deep hands-on practice — I understand HOW to save/load a model and roughly what a deployment setup looks like conceptually, but haven't actually built and shipped a real deployed model yet. Flagging this honestly rather than overstating it, since actual deployment experience is something I expect to build later through real projects, not just from reading about it here.
