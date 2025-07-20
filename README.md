# Test-Sites-
This website Takes in Birthdates and outputs Horoscope Predictions for the Month and lucky numbers 
from flask import Flask, request, jsonify, render_template_string
import datetime
import random

app = Flask(__name__)

# Zodiac data
zodiac_signs = [
    ("Capricorn", (12, 22), (1, 19)),
    ("Aquarius", (1, 20), (2, 18)),
    ("Pisces", (2, 19), (3, 20)),
    ("Aries", (3, 21), (4, 19)),
    ("Taurus", (4, 20), (5, 20)),
    ("Gemini", (5, 21), (6, 20)),
    ("Cancer", (6, 21), (7, 22)),
    ("Leo", (7, 23), (8, 22)),
    ("Virgo", (8, 23), (9, 22)),
    ("Libra", (9, 23), (10, 22)),
    ("Scorpio", (10, 23), (11, 21)),
    ("Sagittarius", (11, 22), (12, 21))
]

monthly_predictions = {
    "Aries": "This month brings bold energy—go after what you want!",
    "Taurus": "Stability is your strength now. Ground yourself and thrive.",
    "Gemini": "Expect communication breakthroughs. Talk it out!",
    "Cancer": "Emotional clarity will guide your decisions. Trust your gut.",
    "Leo": "Your charisma is magnetic. Use it wisely!",
    "Virgo": "Organize your thoughts, and everything else will fall in place.",
    "Libra": "Balance is key. Look for harmony in work and relationships.",
    "Scorpio": "Intensity pays off this month—focus and transform.",
    "Sagittarius": "Adventure calls. Take a risk and explore!",
    "Capricorn": "Your persistence brings progress. Stay steady.",
    "Aquarius": "Innovation is your gift this month—share your ideas.",
    "Pisces": "Dreams and intuition guide you now. Follow them fearlessly."
}

daily_advice = [
    "Smile at a stranger—you never know what it'll spark.",
    "Take a few deep breaths before answering tough questions.",
    "Spend 10 minutes offline. Just be.",
    "Say 'yes' to something you've never tried.",
    "Kindness wins—offer it freely today.",
    "Start with gratitude. It'll change your lens on the day.",
    "Drink more water—your mind will thank you.",
    "Forgive someone today, even quietly."
]

def get_zodiac_sign(birthdate):
    for sign, start, end in zodiac_signs:
        start_date = datetime.date(birthdate.year, *start)
        end_date = datetime.date(birthdate.year if start[0] <= end[0] else birthdate.year + 1, *end)
        if start_date <= birthdate <= end_date:
            return sign
    return "Unknown"

def predict_zodiac(birthdate_str):
    birthdate = datetime.datetime.strptime(birthdate_str, "%Y-%m-%d").date()
    zodiac = get_zodiac_sign(birthdate)
    lucky_numbers = random.sample(range(1, 100), 3)
    advice = random.choice(daily_advice)
    prediction = monthly_predictions.get(zodiac, "Big things are on the horizon.")

    return {
        "Zodiac Sign": zodiac,
        "Monthly Prediction": prediction,
        "Lucky Numbers": lucky_numbers,
        "Daily Advice": advice
    }

# HTML front-end as a string
HTML_PAGE = """
<!DOCTYPE html>
<html>
<head>
    <title>Zodiac Prediction Tool</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 2em;
            background: #f3f3f3;
        }
        input, button {
            padding: 0.5em;
            font-size: 1em;
            margin: 1em;
        }
        #results {
            margin-top: 2em;
            background: #fff;
            padding: 1em;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            max-width: 600px;
            margin-left: auto;
            margin-right: auto;
        }
    </style>
</head>
<body>
    <h1>🔮 Zodiac Prediction Tool</h1>
    <p>Enter your birthday:</p>
    <input type="date" id="birthday">
    <button onclick="getPrediction()">Get My Zodiac Prediction</button>
    <div id="results"></div>

    <script>
        async function getPrediction() {
            const bday = document.getElementById("birthday").value;
            const response = await fetch("/predict", {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({ birthday: bday })
            });
            const data = await response.json();

            document.getElementById("results").innerHTML = `
                <h2>🌟 Your Zodiac Insight 🌟</h2>
                <p><strong>Zodiac Sign:</strong> ${data["Zodiac Sign"]}</p>
                <p><strong>Monthly Prediction:</strong> ${data["Monthly Prediction"]}</p>
                <p><strong>Lucky Numbers:</strong> ${data["Lucky Numbers"].join(", ")}</p>
                <p><strong>Daily Advice:</strong> ${data["Daily Advice"]}</p>
            `;
        }
    </script>
</body>
</html>
"""

@app.route("/")
def index():
    return render_template_string(HTML_PAGE)

@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json()
    birthday = data.get("birthday")
    result = predict_zodiac(birthday)
    return jsonify(result)

if __name__ == "__main__":
    app.run(debug=True)
