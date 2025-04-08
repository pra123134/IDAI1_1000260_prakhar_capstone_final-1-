import streamlit as st
import pandas as pd
import random
import csv
import os
import google.generativeai as genai
import json
from datetime import datetime, timedelta

# Configure API Key securely
if "GOOGLE_API_KEY" in st.secrets:
    api_key = st.secrets["GOOGLE_API_KEY"]
    genai.configure(api_key=api_key)
else:
    st.error("⚠️ API Key is missing. Go to Streamlit Cloud → Settings → Secrets and add your API key.")
    st.stop()

# Generate AI Evaluation for Recipe Name
def evaluate_recipe_name(recipe_name):
    prompt = f"""
    Evaluate the following recipe name for creativity, relevance, and originality:
    "{recipe_name}"
    Provide:
    - A score out of 10
    - A brief reason for the score.
    Return this as a JSON object with keys 'score' and 'reason'.
    """
    try:
        model = genai.GenerativeModel("gemini-1.5-pro")
        response = model.generate_content(prompt)
        response_text = response.text.strip()
        
        # Preprocess the response to remove surrounding backticks or formatting
        if response_text.startswith("```") and response_text.endswith("```"):
            response_text = response_text.strip("```").strip("json").strip()

        # Attempt to parse the response as JSON
        try:
            evaluation = json.loads(response_text)
            if "score" in evaluation and "reason" in evaluation:
                return evaluation
            else:
                return {"score": 0, "reason": "Invalid AI response format."}
        except json.JSONDecodeError:
            return {"score": 0, "reason": f"Failed to parse AI response as JSON: {response_text}"}
    except Exception as e:
        return {"score": 0, "reason": f"AI Error: {str(e)}"}


def save_game_results_to_csv(recipe_names, filename="recipe_contest_results.csv"):
    filepath = os.path.join(os.getcwd(), filename)
    current_date = datetime.today().strftime("%Y-%m-%d")

    try:
        file_exists = os.path.isfile(filepath)  # Check if file exists

        with open(filepath, "a", newline="", encoding="utf-8") as csvfile:
            writer = csv.writer(csvfile)

            # Write header only if file doesn't exist (prevents duplicates)
            if not file_exists:
                writer.writerow(["Chef Name", "Recipe Name", "Score", "Reason", "Ingredients", "Date"])

            # Debug: Print before writing
            print("Debug: Recipe Names Dictionary:", recipe_names)

            # Write new data
            for chef, data in recipe_names.items():
                print(f"Writing Data: {chef}, {data}")  # Debugging
                writer.writerow([chef, data["recipe"], data["score"], data["reason"], data["ingredients"], current_date])

            # Force writing to file
            csvfile.flush()
            csvfile.close()

        print(f"Data successfully saved to {filename}")

    except FileNotFoundError:
        print(f"Error: The file {filename} was not found.")
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        

# Load results from CSV for leaderboard
def load_results_from_csv(filename="recipe_contest_results.csv"):
    filepath = os.path.join(os.getcwd(), filename)
    if os.path.exists(filepath):
        return pd.read_csv(filepath)
    else:
        return pd.DataFrame(columns=["Chef Name", "Recipe Name", "Score", "Reason"])


def display_leaderboard_from_csv(filename):
    filepath = os.path.join(os.getcwd(), filename)
    try:
        df = pd.read_csv(filepath)
        if df.empty:
            st.warning("Leaderboard is empty.")
            return

        # Ensure "Date" column is in datetime format
        df["Date"] = pd.to_datetime(df["Date"], errors="coerce")

        # Ensure "Score" column is numeric
        df["Score"] = pd.to_numeric(df["Score"], errors="coerce")

        # Sort by Date (latest first), then by Score (highest first)
        df = df.sort_values(by=["Date", "Score"], ascending=[False, False])

        st.write("🏆 **Leaderboard** 🏆")
        st.dataframe(df)

    except FileNotFoundError:
        st.error("Leaderboard file not found.")
    except Exception as e:
        st.error(f"An error occurred: {e}")
    
# Function to declare winners
def declare_winners(df):
    if df.empty:
        return "No data available for winners."
    
    df['Date'] = pd.to_datetime(df['Date'])
    df['Week'] = df['Date'].dt.strftime('%Y-%U')  # Year-Week format
    df['Month'] = df['Date'].dt.strftime('%Y-%m')  # Year-Month format
    df['Score'] = pd.to_numeric(df['Score'], errors='coerce')  # Ensure Score is numeric
    
    # Sort before idxmax() to get consistent results in case of ties
    df = df.sort_values(by=['Score', 'Date'], ascending=[False, True])
    
    # Get highest scorer per day, week, and month
    daily_winner = df.loc[df.groupby('Date')['Score'].idxmax().dropna()]
    weekly_winner = df.loc[df.groupby('Week')['Score'].idxmax().dropna()]
    monthly_winner = df.loc[df.groupby('Month')['Score'].idxmax().dropna()]

    # Display Winners
    st.subheader("🏆 Contest Winners")
    
    if not daily_winner.empty:
        st.write("### Daily Winner")
        st.dataframe(daily_winner[['Chef Name', 'Recipe Name', 'Score', 'Date']])
    
    if not weekly_winner.empty:
        st.write("### Weekly Winner")
        st.dataframe(weekly_winner[['Chef Name', 'Recipe Name', 'Score', 'Week']])
    
    if not monthly_winner.empty:
        st.write("### Monthly Winner")
        st.dataframe(monthly_winner[['Chef Name', 'Recipe Name', 'Score', 'Month']])


# Load Data
def load_data():
    file_path = "recipe_contest_results.csv"  # Ensure this file is in the same directory
    df = pd.read_csv(file_path)
    df["Date"] = pd.to_datetime(df["Date"], format="%d-%m-%Y", errors='coerce')  # Convert to datetime safely
    df = df.dropna(subset=["Date"])  # Drop rows with invalid dates
    df["Score"] = pd.to_numeric(df["Score"], errors='coerce')  # Ensure Score is numeric
    df = df.dropna(subset=["Score"])  # Remove rows with NaN scores
    return df

def get_winner(df, period):
    today = datetime.now()
       
    if period == "Day":
        start_date = today.replace(hour=0, minute=0, second=0, microsecond=0)
    elif period == "Week":
        start_date = today - timedelta(days=today.weekday())  # Start of the week
    elif period == "Month":
        start_date = today.replace(day=1)  # Start of the month
    else:
        return "Please select a valid period."
    st.write(start_date)
    recent_data = df[df["Date"] >= start_date]
    st.write(recent_data)
    if recent_data.empty:
        return "No winner available."
    
    winner = recent_data.loc[recent_data["Score"].idxmax()]
    st.write(f"🏆 Winner: {winner['Chef Name']} ({winner['Recipe Name']}) with Score: {winner['Score']}")
    return f"🏆 Winner: {winner['Chef Name']} ({winner['Recipe Name']}) with Score: {winner['Score']}"


# Function to dynamically generate a recipe
def generate_recipe():
    prompt = """
    You are a world-class chef. Generate a recipe with:
    - A unique name for the dish.
    - A list of 5 to 7 essential ingredients.
    Output the response in JSON format:
    {
        "name": "<Dish Name>",
        "ingredients": ["<Ingredient 1>", "<Ingredient 2>", "..."]
    }
    """
    try:
        model = genai.GenerativeModel("gemini-1.5-pro")
        response = model.generate_content(prompt)
        result = response.result.strip()
        return eval(result)  # Convert JSON-like string to Python dictionary
    except Exception as e:
        return {"error": f"AI Error: {str(e)}"}

# Function to evaluate the user's ingredients
def evaluate_ingredients(user_ingredients, correct_ingredients):
    user_ingredients = [item.strip().lower() for item in user_ingredients]
    correct_set = set(correct_ingredients)
    user_set = set(user_ingredients)

    score = len(user_set & correct_set)  # Correct matches
    missed = correct_set - user_set  # Missed ingredients
    extra = user_set - correct_set  # Extra (wrong) ingredients

    return {
        "score": score,
        "missed": list(missed),
        "extra": list(extra),
    }

# Save results to a CSV file
CSV_FILE = "chef_game_results.csv"

def save_results(name, recipe, result):
    file_exists = os.path.isfile(CSV_FILE)
    with open(CSV_FILE, "a", newline="") as file:
        writer = csv.writer(file)
        if not file_exists:
            writer.writerow(["Name", "Recipe", "Score", "Missed Ingredients", "Extra Ingredients"])
        writer.writerow([name, recipe, result["score"], ", ".join(result["missed"]), ", ".join(result["extra"])])

# Streamlit App
#def main():
st.title("🍽️ AI Recipe Name Contest")
st.write("Compete to create the best recipe names!")

# 1: Collect inputs
st.header("1: Enter Chef and Recipe Details")
chefs = []
recipe_data = {}

chef_name = st.text_input("Enter Chef Name")
if chef_name:
    ingredients = random.choice(["chicken, garlic, onion", "quinoa, spinach, avocado", "tofu, soy sauce, ginger"])
    st.write(f"Your ingredients are: **{ingredients}**")
    recipe_name = st.text_input("Enter Your Recipe Name")

    # Evaluate recipe name
    if st.button("Evaluate Recipe Name"):
        with st.spinner("Evaluating recipe..."):
            evaluation = evaluate_recipe_name(recipe_name)
            recipe_data[chef_name] = {
                "recipe": recipe_name,
                "score": evaluation["score"],
                "reason": evaluation["reason"],
            }
            st.success(f"Score: {evaluation['score']}/10")
            st.write(f"Reason: {evaluation['reason']}")

    # Save results
    if recipe_data:
        save_game_results_to_csv(recipe_data,"recipe_contest_results.csv")

# 2: Display Leaderboard
st.header("2: View Leaderboard")

# Show leaderboard file
if st.button("Display the scores"):
    display_leaderboard_from_csv("recipe_contest_results.csv")

st.title("🍽️ Recipe Contest - Winner Announcer")

# Load data
df = load_data()
# Debugging output
#st.write("### Debug: Data Preview")
#st.write(df.head())
period = st.selectbox("Select time period for winner announcement:", ["Day", "Week", "Month"])

if st.button("Show Winner"):
    result = get_winner(df, period)
    st.subheader(result)
    declare_winners(df)

# 3: Guess the Ingradient Game
st.header("3. 👩‍🍳 Chef Game with AI!")
st.write("Test your cooking knowledge by guessing the ingredients of an AI-generated recipe!")

# Generate a recipe name dynamically
st.subheader("AI-Generated Recipe Name")
recipe_data = generate_recipe()

if "error" in recipe_data:
    st.error(recipe_data["error"])
    st.stop()

recipe_name = recipe_data["name"]
correct_ingredients = recipe_data["ingredients"]

st.write(f"**Dish Name:** {recipe_name}")
st.write("Guess the ingredients for this recipe (provide at least 5 to match the AI's choices).")

user_name = st.text_input("Enter your name:")
user_input = st.text_area("Your ingredients (comma-separated):")

if st.button("Submit"):
    if not user_name.strip() or not user_input.strip():
        st.error("⚠️ Please provide your name and ingredients!")
    else:
        user_ingredients = user_input.split(",")
        result = evaluate_ingredients(user_ingredients, correct_ingredients)

        st.subheader("🎉 Results")
        st.write(f"**Score:** {result['score']} / {len(correct_ingredients)}")
        st.write(f"**Missed Ingredients:** {', '.join(result['missed']) if result['missed'] else 'None'}")
        st.write(f"**Extra Ingredients:** {', '.join(result['extra']) if result['extra'] else 'None'}")

        # Save results to a CSV file
        save_results(user_name, recipe_name, result)
        st.success("✅ Your results have been saved!")

        # Display CSV download link
        if os.path.isfile(CSV_FILE):
            with open(CSV_FILE, "r") as file:
                st.download_button("📥 Download Results", file.read(), file_name=CSV_FILE)
