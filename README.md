import streamlit as st
import google.generativeai as genai

# Config Gemini
genai.configure(api_key="AIzaSyAOUeQcJrcqdiNb0HR3htpDzWQnhD5by04")

# Load the model
model = genai.GenerativeModel(model_name="gemini-1.5-pro")

# Page Config
st.set_page_config(page_title="Interactive City Explorer", page_icon="🗺️", layout="wide")

# Styles
st.markdown(
    """
    <style>
        body {
            background-color: #fafafa;
            color: #333;
            font-family: 'Helvetica', sans-serif;
        }
        .stMarkdown h1 {
            text-align: center;
            color: #004d40;
        }
        .stButton>button {
            background-color: #00897b;
            color: white;
            border: none;
            border-radius: 5px;
            font-size: 14px;
            padding: 8px 16px;
        }
        .stButton>button:hover {
            background-color: #004d40;
        }
        .query-button {
            margin: 5px 0;
        }
    </style>
    """,
    unsafe_allow_html=True,
)

# Header
st.markdown("<h1>🗺️ Explore Cities Around the World</h1>", unsafe_allow_html=True)
st.markdown(
    "<p style='text-align: center;'>Discover famous places, cultural insights, best markets, and more for your favorite cities.</p>",
    unsafe_allow_html=True,
)

# Sidebar for quick query options
st.sidebar.markdown("## 🔍 Quick Queries")
quick_queries = [
    "Best food to try 🍜",
    "Hidden gems 🌟",
    "Famous landmarks 🗼",
    "Best shopping spots 🛍️",
    "Cultural must-sees 🎭",
]
for query in quick_queries:
    if st.sidebar.button(query, key=query):
        st.session_state["user_input"] = query

# User Input
st.markdown("### 🌆 Enter City Details:")
city_name = st.text_input("City Name", placeholder="E.g., Tokyo, Paris, Cairo")
topic = st.selectbox(
    "What do you want to know?",
    ["Highlights", "Famous Places", "Best Markets", "Cultural Insights", "Other"],
)

custom_query = st.text_area(
    "Ask your specific query about the city:", placeholder="E.g., What are the best attractions in this city?"
)

# Button to Process Query
if st.button("Get Information"):
    if city_name.strip() == "":
        st.error("Please provide a city name.")
    else:
        # Generate prompt
        prompt = f"""
        You are a city exploration assistant. Provide detailed and engaging information based on the following:
        - City: {city_name}
        - Topic: {topic}
        - User Query: {custom_query if custom_query else 'Provide general information about this topic.'}

        Include details about:
        - Famous landmarks, key highlights
        - Best markets and shopping areas
        - Cultural or historical significance
        """
        with st.spinner("Fetching city insights..."):
            try:
                response = model.generate_content(prompt)
                sections = response.text.strip().split("\n\n")
                for section in sections:
                    st.markdown(f"### {section.strip()}")
            except Exception as e:
                st.error(f"Failed to retrieve information: {e}")

# Additional Interactive Options
st.markdown("### 🏙️ Explore More:")
cols = st.columns(4)
options = ["Famous Places", "Best Food", "Top Markets", "Hidden Gems"]
for i, option in enumerate(options):
    with cols[i]:
        if st.button(option, key=option):
            st.session_state["user_input"] = option
            if city_name.strip():
                st.write(f"Fetching {option.lower()} for {city_name}...")
                # Simulated output
                st.markdown(f"**{option} in {city_name} will appear here.**")
            else:
                st.error("Please enter a city name.")

# Save to History
if "history" not in st.session_state:
    st.session_state["history"] = {}

if st.button("Save to History"):
    if city_name:
        if city_name not in st.session_state["history"]:
            st.session_state["history"][city_name] = []
        st.session_state["history"][city_name].append({"topic": topic, "query": custom_query})
        st.success(f"Saved information for {city_name}!")

# Display History
if st.session_state["history"]:
    st.markdown("### 🕰️ City Query History")
    for city, queries in st.session_state["history"].items():
        st.markdown(f"#### {city}")
        for entry in queries:
            st.markdown(f"- **Topic**: {entry['topic']} | **Query**: {entry['query']}")
