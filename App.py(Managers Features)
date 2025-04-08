import streamlit as st
import google.generativeai as genai

# ✅ Configure API Key securely
if "GOOGLE_API_KEY" in st.secrets:
    api_key = st.secrets["GOOGLE_API_KEY"]
    genai.configure(api_key=api_key)
else:
    st.error("⚠️ API Key is missing. Go to Streamlit Cloud → Settings → Secrets and add your API key.")
    st.stop()

# ✅ AI Response Generator
def get_ai_response(prompt, fallback_message="⚠️ AI response unavailable. Please try again later."):
    try:
        model = genai.GenerativeModel("gemini-1.5-pro")
        response = model.generate_content(prompt)
        return response.text.strip() if hasattr(response, "text") and response.text.strip() else fallback_message
    except Exception as e:
        return f"⚠️ AI Error: {str(e)}\n{fallback_message}"

# ✅ Streamlit UI Configuration
st.set_page_config(page_title="Smart Restaurant Menu Management App", layout="wide")

st.title("🍽️ Smart Restaurant Menu Management with Gemini 1.5 Pro")
st.write("🚀 Manage events, recommend menus, and optimize leftovers using GenAI.")

# 🎯 **Event Manager**
st.header("🎉 Event Manager")

occasion = st.text_input("🎊 Occasion (e.g., Birthday, Anniversary, Corporate Event)")
people = st.number_input("👥 Number of Guests", min_value=1, value=2)
cuisine_type = st.selectbox("🍱 Cuisine Preference", ["Veg", "Non-Veg", "Vegan", "Mixed"])
drink_type = st.selectbox("🍹 Drink Preference", ["Soft Drinks", "Mocktails", "Cocktails", "Beer", "Wine"])
budget = st.text_input("💰 Budget Range (e.g., $100-$300)")

if st.button("✨ Generate Event Plan"):
    if not all([occasion, people, cuisine_type, drink_type, budget]):
        st.error("⚠️ Please fill in all fields before generating a recommendation.")
    else:
        prompt = f"""
        Create an event plan for:
        - Occasion: {occasion}
        - Guests: {people}
        - Cuisine: {cuisine_type}
        - Drinks: {drink_type}
        - Budget: {budget}
        
        Include:
        - Menu recommendations
        - Decoration style
        - Discount strategies
        - Marketing slogan
        - Instagram caption with trending hashtags
        - Sustainability tips
        """
        st.text_area("📋 Event Plan:", get_ai_response(prompt), height=300)

# 🍽️ **Food Menu Recommendation**
st.header("🍴 Food Menu Recommendation")

meal_type = st.selectbox("🍔 Meal Type", ["Breakfast", "Lunch", "Dinner", "Snack"])
dietary_pref = st.selectbox("🥗 Dietary Preference", ["Vegetarian", "Non-Vegetarian", "Vegan", "Keto", "Gluten-Free"])
spice_level = st.selectbox("🌶️ Spice Level", ["Mild", "Medium", "Spicy", "Extra Spicy"])

if st.button("🔍 Recommend Food Menu"):
    if not all([meal_type, dietary_pref, spice_level]):
        st.error("⚠️ Please fill in all fields before generating a recommendation.")
    else:
        prompt = f"""
        Generate a food menu recommendation for:
        - Meal: {meal_type}
        - Dietary Preference: {dietary_pref}
        - Spice Level: {spice_level}
        
        Include:
        - Appetizers
        - Main Course
        - Side Dishes
        - Drinks
        - Desserts
        - AI-powered pairing suggestions
        """
        st.text_area("🍽️ Menu Recommendation:", get_ai_response(prompt), height=300)

# 🥗 **Leftover Management**
st.header("🥡 Leftover Management")

leftover_type = st.selectbox("🥩 Leftover Type", ["Meat", "Vegetables", "Dairy", "Grains", "Fruits"])
quantity = st.number_input("📦 Quantity (in kg)", min_value=0.1, value=1.0)
shelf_life = st.number_input("⏳ Shelf Life (in days)", min_value=1, value=3)

if st.button("♻️ Optimize Leftover Usage"):
    if not all([leftover_type, quantity, shelf_life]):
        st.error("⚠️ Please fill in all fields before generating a recommendation.")
    else:
        prompt = f"""
        Optimize the management of:
        - Leftover: {leftover_type}
        - Quantity: {quantity} kg
        - Shelf Life: {shelf_life} days
        
        Include:
        - Creative leftover recipes
        - Preservation techniques
        - Donation suggestions
        - Sustainability strategies
        """
        st.text_area("♻️ Leftover Optimization:", get_ai_response(prompt), height=300)

# ✅ Footer
st.write("🚀 Powered by Gemini 1.5 Pro with GenAI")
