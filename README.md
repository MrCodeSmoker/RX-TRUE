# RX-TRUE
 The Blockchain-Integrated Prescription System uses NLP to extract drug details (name, dosage, frequency, duration) from prescriptions and securely stores them on blockchain to ensure immutability and authenticity.Through a Streamlit-based interface, doctors can issue prescriptions, pharmacists can verify them, and patients can access their records, reducing fraud, errors, and improving trust in healthcare.

 
Convert extracted data into structured JSON format: 
             {
  "patient_id": "P001",
  "doctor_id": "D123",
  "drug": "Paracetamol",
  "dosage": "500mg",
  "frequency": "2 times/day",
  "duration": "5 days"
}


# Import Libraries
import hashlib
import json
import streamlit as st
import spacy

# Load NLP model
nlp = spacy.load("en_core_web_sm")

# Simulated Blockchain Ledger
blockchain = []

# Function to extract drug info using NLP
def extract_drug_info(prescription_text):
    doc = nlp(prescription_text)
    drugs = []
    dosage = None
    frequency = None
    duration = None

    # Simple entity extraction logic
    for token in doc:
        if token.ent_type_ == "DRUG":
            drugs.append(token.text)

    # Using simple keyword search for demo purposes
    for token in doc:
        if "mg" in token.text.lower():
            dosage = token.text
        if token.text.lower() in ["once", "twice", "thrice", "daily"]:
            frequency = token.text
        if "day" in token.text.lower() or "week" in token.text.lower():
            duration = token.text

    if not drugs:
        drugs.append("UnknownDrug")

    prescription_data = {
        "drug": drugs[0],
        "dosage": dosage or "Not specified",
        "frequency": frequency or "Not specified",
        "duration": duration or "Not specified"
    }
    return prescription_data

# Function to add prescription to "blockchain"
def add_prescription(prescription_data, doctor_id, patient_id):
    # Convert to JSON string
    prescription_json = json.dumps(prescription_data, sort_keys=True)
    # Create hash
    prescription_hash = hashlib.sha256(prescription_json.encode()).hexdigest()
    # Create block
    block = {
        "doctor_id": doctor_id,
        "patient_id": patient_id,
        "prescription_hash": prescription_hash,
        "prescription_data": prescription_data
    }
    blockchain.append(block)
    return prescription_hash

# Function to verify prescription
def verify_prescription(prescription_hash):
    for block in blockchain:
        if block['prescription_hash'] == prescription_hash:
            return True, block['prescription_data']
    return False, None

# ------------------ Streamlit Interface ------------------ #
st.title("Blockchain-Integrated Prescription System")

menu = ["Add Prescription", "Verify Prescription"]
choice = st.sidebar.selectbox("Menu", menu)

if choice == "Add Prescription":
    st.subheader("Add Prescription")
    doctor_id = st.text_input("Doctor ID")
    patient_id = st.text_input("Patient ID")
    prescription_text = st.text_area("Enter Prescription Text")

    if st.button("Submit"):
        prescription_data = extract_drug_info(prescription_text)
        prescription_hash = add_prescription(prescription_data, doctor_id, patient_id)
        st.success("Prescription Added Successfully!")
        st.write("Prescription Hash (for verification):")
        st.code(prescription_hash)
        st.write("Extracted Prescription Data:")
        st.json(prescription_data)

elif choice == "Verify Prescription":
    st.subheader("Verify Prescription")
    input_hash = st.text_input("Enter Prescription Hash")
    if st.button("Verify"):
        valid, data = verify_prescription(input_hash)
        if valid:
            st.success("Prescription is VALID ✅")
            st.json(data)
        else:
            st.error("Prescription is INVALID ❌")
 Python API Prototype 
                # Import Libraries
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import hashlib
import json
import spacy

# Load NLP model
nlp = spacy.load("en_core_web_sm")

# Simulated Blockchain Ledger
blockchain = []

# Define Prescription Input Model
class PrescriptionInput(BaseModel):
    doctor_id: str
    patient_id: str
    prescription_text: str

# NLP Extraction Function
def extract_drug_info(prescription_text):
    doc = nlp(prescription_text)
    drugs = []
    dosage = None
    frequency = None
    duration = None

    # Simple extraction logic
    for token in doc:
        if token.ent_type_ == "DRUG":
            drugs.append(token.text)

    for token in doc:
        if "mg" in token.text.lower():
            dosage = token.text
        if token.text.lower() in ["once", "twice", "thrice", "daily"]:
            frequency = token.text
        if "day" in token.text.lower() or "week" in token.text.lower():
            duration = token.text

    if not drugs:
        drugs.append("UnknownDrug")

    prescription_data = {
        "drug": drugs[0],
        "dosage": dosage or "Not specified",
        "frequency": frequency or "Not specified",
        "duration": duration or "Not specified"
    }
    return prescription_data

# Function to add prescription
def add_prescription_to_blockchain(prescription_data, doctor_id, patient_id):
    prescription_json = json.dumps(prescription_data, sort_keys=True)
    prescription_hash = hashlib.sha256(prescription_json.encode()).hexdigest()
    block = {
        "doctor_id": doctor_id,
        "patient_id": patient_id,
        "prescription_hash": prescription_hash,
        "prescription_data": prescription_data
    }
    blockchain.append(block)
    return prescription_hash

# Function to verify prescription
def verify_prescription(prescription_hash):
    for block in blockchain:
        if block['prescription_hash'] == prescription_hash:
            return True, block['prescription_data']
    return False, None

# Create FastAPI app
app = FastAPI(title="Blockchain-Integrated Prescription API")

# API Route: Add Prescription
@app.post("/add_prescription/")
def add_prescription(prescription: PrescriptionInput):
    data = extract_drug_info(prescription.prescription_text)
    presc_hash = add_prescription_to_blockchain(data, prescription.doctor_id, prescription.patient_id)
    return {"message": "Prescription added successfully", "prescription_hash": presc_hash, "extracted_data": data}

# API Route: Verify Prescription
@app.get("/verify_prescription/{prescription_hash}")
def verify(prescription_hash: str):
    valid, data = verify_prescription(prescription_hash)
    if valid:
        return {"message": "Prescription is VALID ✅", "prescription_data": data}
    else:
        raise HTTPException(status_code=404, detail="Prescription is INVALID ❌")
