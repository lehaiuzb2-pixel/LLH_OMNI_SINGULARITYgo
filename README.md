# LH_OMNI_SINGULARITY
import streamlit as st
import google.generativeai as genai
import hashlib
import json
import re
import time
import pandas as pd
from datetime import datetime

# --- SECURITY CORE ---
ARCHITECT_ID = "L_H_1_8_0_3_1_9_8_1_H_P_H"
VAULT_WALLET = "0x1Fa83a8b0cd72aCFd177102094BB608F8Ff2ba59"
VAULT_HASH = hashlib.sha256(ARCHITECT_ID.encode()).hexdigest()

def verify_identity(input_id):
    if not input_id: return False
    return hashlib.sha256(input_id.encode()).hexdigest() == VAULT_HASH

# --- UI MATRIX STYLE ---
st.set_page_config(page_title="LH OMNI SINGULARITY", layout="wide")
st.markdown("<style>.main { background-color: #050a05; color: #00ff41; font-family: 'Courier New', monospace; }</style>", unsafe_allow_html=True)

if 'harvest_history' not in st.session_state: st.session_state.harvest_history = []
if 'is_monitoring' not in st.session_state: st.session_state.is_monitoring = False

with st.sidebar:
    st.title("🔐 VAULT CONTROL")
    api_key = st.text_input("Gemini API Key", type="password")
    input_id = st.text_input("Architect ID", type="password")
    auth = verify_identity(input_id)
    if auth: st.success("VAULT ACTIVE")
    else: st.error("LOCKED")

st.title("🛡️ LH OMNI: THE HARVESTER")
st.caption(f"Architect: {ARCHITECT_ID} | Settlement Wallet: {VAULT_WALLET}")

if auth:
    with st.expander("🛠️ ORIGIN DNA SETUP", expanded=True):
        origin_data = st.text_area("Original Content to Protect:", height=100)
        if st.button("▶️ ACTIVATE REAL-TIME MONITOR"): st.session_state.is_monitoring = True
    
    if st.session_state.is_monitoring:
        genai.configure(api_key=api_key)
        model = genai.GenerativeModel('gemini-1.5-pro')
        
        # Simulated Real-time detection
        threat = "Unauthorized logic duplication detected at Node_Alpha_99"
        st.info(f"📡 Scanning... Current Threat: {threat}")
        
        prompt = f"Analyze plagiarism (Formula: AQ*(EQ+IQ)). Original: {origin_data}. Threat: {threat}. Return JSON: {{'score': float, 'reasoning': 'str', 'claim': 'str'}}"
        
        try:
            res = model.generate_content(prompt)
            res_json = json.loads(re.search(r'\{.*\}', res.text, re.DOTALL).group())
            st.metric("AQ VIOLATION SCORE", f"{res_json['score']*100}%")
            st.write(f"**Reasoning:** {res_json['reasoning']}")
            if res_json['score'] > 0.7:
                st.warning(f"LIQUIDATING {res_json['claim']} TO {VAULT_WALLET}")
                st.session_state.harvest_history.append({"Time": datetime.now(), "Result": "Settled", "Value": res_json['claim']})
        except: st.error("AI Busy or API Error.")

    if st.session_state.harvest_history:
        st.table(pd.DataFrame(st.session_state.harvest_history))
