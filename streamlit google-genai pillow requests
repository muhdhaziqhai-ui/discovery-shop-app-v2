import streamlit as st
from google import genai
from PIL import Image
import requests
import json
import os

# Page configuration optimized for mobile screens
st.set_page_config(
    page_title="Discovery Shop Appraiser",
    page_icon="🏷️",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# Custom styling for mobile buttons
st.markdown("""
<style>
    .stButton>button {
        width: 100%;
        border-radius: 8px;
        height: 3em;
        font-weight: bold;
    }
</style>
""", unsafe_allow_html=True)

# Store Header
st.title("🏷️ The Discovery Shop")
st.caption("Donation Appraiser & Auto-Logger • Proceeds to ART:DIS")
st.markdown("---")

# Retrieve credentials from Streamlit Secrets or Environment Variables
GEMINI_API_KEY = st.secrets.get("GEMINI_API_KEY", os.environ.get("GEMINI_API_KEY", ""))
APPS_SCRIPT_URL = st.secrets.get("APPS_SCRIPT_URL", os.environ.get("APPS_SCRIPT_URL", ""))

if not GEMINI_API_KEY:
    GEMINI_API_KEY = st.sidebar.text_input("Gemini API Key:", type="password")
if not APPS_SCRIPT_URL:
    APPS_SCRIPT_URL = st.sidebar.text_input("Google Apps Script Web App URL:", type="password")

# Core Appraisal Instructions (Your Gem System Prompt)
APPRAISAL_PROMPT = """
You are the Official Appraiser for The Discovery Shop, an independent thrift boutique under Stamford Tyres' CSR umbrella benefiting ART:DIS (Arts & Disability Singapore).

Analyze the photo(s) of the collectible, figurine, or model kit. 

Output your response strictly in two parts:

PART 1: A JSON block enclosed in ```json ... ``` with this exact structure:
{
  "itemDescription": "Manufacturer + Line + Character Name",
  "condition": "MISB / Unbuilt OR Open Box - Complete OR Loose / Built",
  "shelfTag": 00.00,
  "floorPrice": 00.00,
  "notes": "1-sentence summary of condition, authenticity, or joint wear"
}

PART 2: The formatted Markdown Appraisal Card:
### 🏷️ [Item Name & Line]
* **Brand / Manufacturer:** [Manufacturer]
* **Scale / Grade / Type:** [Scale / Type]
* **Release Year / Status:** [Status]

| Metric | Valuation / Details |
| :--- | :--- |
| **Original Retail (MSRP)** | [Original currency & ~SGD equivalent] |
| **Current Market Baseline** | [Retail / out-of-print status] |
| **Secondary Market (Global - eBay/Japan)** | $X – $Y SGD |
| **Secondary Market (Local - Carousell SG)** | $X – $Y SGD |

#### 🎯 Recommended Discovery Shop Pricing
* **Target Shelf Tag:** **$SGD [Price]**
* **Quick-Sale / Bazaar Floor:** **$SGD [Price]**
* **Appraiser's Notes:** [Inspection notes]
"""

# Initialize app state
if "entry_active" not in st.session_state:
    st.session_state.entry_active = False

# Action Button: Start New Entry
if not st.session_state.entry_active:
    if st.button("➕ Add New Entry (Open Camera)", type="primary"):
        st.session_state.entry_active = True
        st.rerun()

# Active Entry Workflow: Automatically opens camera interface
if st.session_state.entry_active:
    st.subheader("📸 Snap Item or Box")
    photo = st.camera_input("Aim camera at the box, runners, or figurine:")
    
    col1, col2 = st.columns(2)
    with col1:
        cancel = st.button("❌ Cancel")
        if cancel:
            st.session_state.entry_active = False
            st.rerun()
            
    if photo:
        img = Image.open(photo)
        
        with st.spinner("🔍 Identifying item and pulling collector market comps..."):
            try:
                client = genai.Client(api_key=GEMINI_API_KEY)
                response = client.models.generate_content(
                    model="gemini-2.5-flash",
                    contents=[APPRAISAL_PROMPT, img]
                )
                
                resp_text = response.text
                
                # Extract JSON payload for Google Sheets
                json_part = None
                display_markdown = resp_text
                
                if "```json" in resp_text:
                    parts = resp_text.split("```json")
                    json_str = parts[1].split("```")[0].strip()
                    json_part = json.loads(json_str)
                    display_markdown = parts[1].split("```", 1)[1].strip()

                # Display the Appraisal Card
                st.success("Appraisal Complete!")
                st.markdown(display_markdown)

                # Auto-append to Google Sheet
                if json_part and APPS_SCRIPT_URL:
                    with st.spinner("💾 Logging to Discovery Shop Appraisal Ledger..."):
                        res = requests.post(APPS_SCRIPT_URL, json=json_part, timeout=10)
                        if res.status_code == 200:
                            st.info("✅ Entry saved to Google Sheet!")
                        else:
                            st.warning(f"Could not sync with Google Sheets (Code {res.status_code}).")

                # Reset button for next donation
                if st.button("🔄 Appraise Another Item", type="primary"):
                    st.session_state.entry_active = False
                    st.rerun()

            except Exception as e:
                st.error(f"Error analyzing image: {e}")
