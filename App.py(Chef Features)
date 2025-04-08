import google.generativeai as genai
import PyPDF2
from PIL import Image
import io
import streamlit as st

# Configure Gemini API
if "GOOGLE_API_KEY" in st.secrets:
    api_key = st.secrets["GOOGLE_API_KEY"]
    genai.configure(api_key=api_key)
else:
    st.error("⚠️ API Key is missing. Go to Streamlit Cloud → Settings → Secrets and add your API key.")
    st.stop()

# Load the appropriate Gemini model
model = genai.GenerativeModel("gemini-1.5-pro")

# Function to extract text from PDF
def extract_text_from_pdf(pdf_file):
    pdf_reader = PyPDF2.PdfReader(pdf_file)
    text = ""
    for page in pdf_reader.pages:
        text += page.extract_text()
    return text

# General recipe generation function
def generate_recipe(user_input=None, image=None, pdf_text=None):
    prompt = f"""
    You are an expert chef. Based on the following inputs, generate a detailed recipe:
    - User Input: {user_input}
    - PDF Content (if provided): {pdf_text if pdf_text else 'None'}
    Provide a recipe that includes:
    - Ingredients list
    - Step-by-step instructions
    - Cooking time and serving size
    - Any dietary considerations mentioned in the input
    """

    if image:
        img_byte_arr = io.BytesIO()
        image.save(img_byte_arr, format='PNG')
        img_data = img_byte_arr.getvalue()
        response = model.generate_content([prompt, {"mime_type": "image/png", "data": img_data}])
    else:
        response = model.generate_content(prompt)

    return response.text

# Seasonal recipe generation function
def generate_seasonal_recipe(ingredients, season, cuisine):
    model = genai.GenerativeModel("gemini-1.5-flash")
    prompt = (f"Create an innovative recipe using seasonal ingredients: {season}, "
              f"local specialties: {ingredients}, and in the style of {cuisine} cuisine. "
              "Provide a creative dish name, key ingredients, and a step-by-step preparation method.")

    response = model.generate_content(prompt)
    return response.text.strip() if response else "No recipe generated. Try again."

# Streamlit App
st.title("👨‍🍳 AI Chef Recipe Generator")
st.write("Generate delicious recipes from names, images, PDFs, or seasonal ingredients!")

options = ["Recipe by Name", "Recipe from Image", "Recipe from PDF", "Seasonal Ingredients Recipe", "Leftover Ingredients Recipe"]
choice = st.selectbox("Choose your recipe type:", options)

if choice == "Recipe by Name":
    user_input = st.text_input("Enter your dietary preferences, cuisine type, or ingredients:")
    if st.button("Generate Recipe from Name", key="btn_name"):
        recipe = generate_recipe(user_input=user_input)
        st.subheader("🍽️ AI-Generated Recipe")
        st.write(recipe)

elif choice == "Recipe from Image":
    uploaded_image = st.file_uploader("Upload an image of ingredients", type=["png", "jpg", "jpeg"])
    if uploaded_image:
        image = Image.open(uploaded_image)
        st.image(image, caption="Uploaded Image", use_column_width=True)
        if st.button("Generate Recipe from Image", key="btn_image"):
            recipe = generate_recipe(image=image)
            st.subheader("🍽️ AI-Generated Recipe")
            st.write(recipe)

elif choice == "Recipe from PDF":
    uploaded_pdf = st.file_uploader("Upload a PDF file", type=["pdf"])
    if uploaded_pdf:
        pdf_text = extract_text_from_pdf(uploaded_pdf)
        st.text_area("Extracted Text from PDF:", pdf_text, height=150)
        if st.button("Generate Recipe from PDF", key="btn_pdf"):
            recipe = generate_recipe(pdf_text=pdf_text)
            st.subheader("🍽️ AI-Generated Recipe")
            st.write(recipe)

elif choice == "Seasonal Ingredients Recipe":
    season = st.selectbox("Select the current season:", ["Spring", "Summer", "Autumn", "Winter"])
    cuisine = st.text_input("Preferred cuisine style (e.g., Italian, Indian, Fusion):")
    ingredients = st.text_input("Enter local specialties (comma-separated):")
    if st.button("Generate Seasonal Recipe", key="btn_seasonal"):
        recipe = generate_seasonal_recipe(ingredients, season, cuisine)
        st.subheader("🍲 AI-Generated Recipe")
        st.write(recipe)

elif choice == "Leftover Ingredients Recipe":
    user_input = st.text_input("Enter leftover ingredients and preferred cuisine:")
    if st.button("Generate Recipe from Leftovers", key="btn_leftover"):
        recipe = generate_recipe(user_input=user_input)
        st.subheader("🍽️ AI-Generated Recipe")
        st.write(recipe)
