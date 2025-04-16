import streamlit as st
import random #to generate random numbers for our game
import openai #to interact with openai
import altair as alt #to create interactive bar chart for the stats part
import dotenv 
from dotenv import load_dotenv #to load the .env file
import os


load_dotenv() #to load the .env file

api_key = os.getenv("OPENAI_API_KEY")
# Seting up OpenAI key for our project
openai.api_key = api_key
model = "gpt-4o-mini"

# Set up Streamlit pages
st.set_page_config(page_title="Guessing Game with OpenAI", page_icon="🎲")

# Initialize stats tracking in session state
if "stats" not in st.session_state: #st.session_state: a special dictionary-like object in streamlit
    st.session_state.stats = {
        "games_played": 0,
        "total_guesses": 0,
        "guesses_per_game": [],
        "quality_scores_per_game": [],  # Track the quality scores for each game in the stats section
    }

# Helper function for OpenAI hints
def generate_openai_hint(guess, number_to_guess):
    try:
        system_message = (
            "You are an assistant for a number guessing game. "
            "Your job is to give helpful hints, such as 'higher,' 'lower,' "
            "or 'you're very close,' to help the user guess the number."
        )
        question = f"The user guessed {guess}. The correct number is {number_to_guess}. Provide a helpful, accurate and friendly hint."

        response = openai.ChatCompletion.create(
            model=model,
            messages=[
                {"role": "system", "content": system_message},
                {"role": "user", "content": question},
            ],
            temperature=0.2,  # Controlling the randomness, better to have it up tp 0.2 for more rational answers
        )
        return response["choices"][0]["message"]["content"]
    except Exception as e:
        return f"Could not fetch a hint due to an error: {e}"

# calculating the qualaity of the guess
def calculate_quality_score(guess, correct_number):
    return max(0, 100 - abs(correct_number - guess)) #I used max to never have negative score, minimum score like this would be 0, also the score is out of 100 here. that could also be out of e.g 50

# "Play" Page
if "page" not in st.session_state:
    st.session_state.page = "play"

st.sidebar.title("Navigate")
if st.sidebar.button("Play"):
    st.session_state.page = "play"
elif st.sidebar.button("Stats"):
    st.session_state.page = "stats"

# The Game Logic
if st.session_state.page == "play":
    st.title("🎲 Guess the Number with OpenAI Hints!")

    if "number_to_guess" not in st.session_state:
        st.session_state.number_to_guess = random.randint(1, 100) #for generating a random number using the random library
        st.session_state.attempts = 0
        st.session_state.quality_scores = []  # Track quality of guesses in the game

    guess = st.text_input("Enter your guess (1-100):", key="guess_input")
    if st.button("Submit Guess"):
        if not guess.isdigit():
            st.write("Please enter a valid number!")
        else:
            guess = int(guess)
            st.session_state.attempts += 1
            quality_score = calculate_quality_score(guess, st.session_state.number_to_guess)
            st.session_state.quality_scores.append(quality_score)

            if guess < st.session_state.number_to_guess:
                hint = generate_openai_hint(guess, st.session_state.number_to_guess)
                st.write(f"Your hint from Open Ai ^_^: {hint}")
            elif guess > st.session_state.number_to_guess:
                hint = generate_openai_hint(guess, st.session_state.number_to_guess)
                st.write(f"Your hint from Open Ai ^_^: {hint}")
            else:
                st.write(f"🎉 Congrats! You guessed it in {st.session_state.attempts} attempts.")
                # Update stats
                st.session_state.stats["games_played"] += 1
                st.session_state.stats["total_guesses"] += st.session_state.attempts
                st.session_state.stats["guesses_per_game"].append(st.session_state.attempts)
                avg_quality = sum(st.session_state.quality_scores) / len(st.session_state.quality_scores)
                st.session_state.stats["quality_scores_per_game"].append(avg_quality)
                # Reset game
                st.session_state.number_to_guess = random.randint(1, 100)
                st.session_state.attempts = 0
                st.session_state.quality_scores = []

# "Stats" Page
elif st.session_state.page == "stats":
    st.title("📊 Game Stats")

    # Retrieve stats from session state
    games_played = st.session_state.stats["games_played"]
    total_guesses = st.session_state.stats["total_guesses"]
    guesses_per_game = st.session_state.stats["guesses_per_game"]
    quality_scores_per_game = st.session_state.stats["quality_scores_per_game"]

    # Display stats
    st.write(f"**Games Played:** {games_played}")
    if games_played > 0:
        avg_guesses = total_guesses / games_played
        avg_quality = sum(quality_scores_per_game) / games_played
        st.write(f"**Average Guesses per Game:** {avg_guesses:.2f}")
        st.write(f"**Average Quality Score per Game:** {avg_quality:.2f}")
    else:
        st.write("**Average Guesses per Game:** No games played yet.")
        st.write("**Average Quality Score per Game:** No games played yet.")

    # Bar chart for guesses per game
    if games_played > 0:
        data = {
            "Game": list(range(1, len(guesses_per_game) + 1)),
            "Guesses": guesses_per_game,
            "Quality Score": quality_scores_per_game,
        }
        guesses_chart = alt.Chart(alt.Data(values=data)).mark_bar().encode(
            x=alt.X("Game:O", title="Game Number", axis=alt.Axis(labelAngle=360)),
            y=alt.Y("Guesses:Q", title="Number of Guesses"),
        ).properties(
            title="Guesses per Game"
        )
        st.altair_chart(guesses_chart, use_container_width=True)

        # Line chart for quality scores
        quality_chart = alt.Chart(alt.Data(values=data)).mark_line(point=True).encode(
            x=alt.X("Game:O", title="Game Number", axis=alt.Axis(labelAngle=360)),
            y=alt.Y("Quality Score:Q", title="Average Quality Score"),
        ).properties(
            title="Quality Score per Game"
        )
        st.altair_chart(quality_chart, use_container_width=True)
    else:
        st.write("No games played yet to display stats.")

