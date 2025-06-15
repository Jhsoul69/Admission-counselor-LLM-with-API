# Admission-counselor-LLM-with-API
AI-powered Admission Counselor using LLMs and APIs to guide students in course selection by analyzing preferences, eligibility, and real-time university data.

# AI-Assisted Admission Counselor Tool
# ----------------------------------------------------
# This tool helps students find programs based on their interests,
# strengths, and preferences. It uses a simple SWOT analysis,
# matches relevant programs, and generates advice using an AI model
# via OpenRouter API.

import requests

# Get basic user profile details like interests, strengths, and preferences
def get_student_profile():
    print("Enter your academic and personal preferences below:")
    academic_interests = input("Your Academic Interests (e.g. AI, CS): ").strip()
    strengths = input("Your Strengths (e.g. problem-solving, math): ").strip()
    preferences = input("Your Preferences (e.g. India, USA): ").strip()

    if not academic_interests or not strengths or not preferences:
        print("All fields are mandatory. Please restart and provide complete details.")
        exit()

    return {
        "academic_interests": academic_interests.lower(),
        "strengths": strengths.lower(),
        "preferences": preferences.lower()
    }

# Basic rule-based SWOT generator using keywords in strengths
def perform_swot(profile):
    swot = {"Strengths": [], "Weaknesses": [], "Opportunities": [], "Threats": []}
    s = profile["strengths"]

    if "math" in s: swot["Strengths"].append("Strong quantitative ability")
    if "communication" in s: swot["Strengths"].append("Good communication skills")
    if "leadership" in s: swot["Strengths"].append("Leadership experience")

    if not swot["Strengths"]:
        swot["Weaknesses"].append("No specific strengths identified")

    swot["Opportunities"].extend([
        "Apply to top STEM or business programs",
        "Showcase leadership and projects in your field"
    ])
    swot["Threats"].append("High competition for top universities")

    return swot

# Predefined program list with keywords for matching user interests
programs = [
    {"name": "B.Tech in AI - IIT Hyderabad", "keywords": ["ai", "artificial intelligence"], "country": "India"},
    {"name": "BSc Computer Science - MIT", "keywords": ["computer science", "cs"], "country": "USA"},
    {"name": "BE Data Science - BITS Pilani", "keywords": ["data", "science"], "country": "India"},
    {"name": "BS Business Analytics - NYU", "keywords": ["business", "analytics"], "country": "USA"},
    {"name": "BBA - IIM Indore", "keywords": ["business", "management"], "country": "India"},
    {"name": "B.Tech CSE - NIT Trichy", "keywords": ["computer", "cse"], "country": "India"},
    {"name": "BS Statistics - UC Berkeley", "keywords": ["statistics", "math"], "country": "USA"},
    {"name": "BS Robotics - Carnegie Mellon", "keywords": ["robotics", "ai"], "country": "USA"}
]

# Match user profile with available programs

def match_programs(profile):
    interests = [i.strip() for i in profile["academic_interests"].split(",")]
    prefs = profile["preferences"]
    matches = []

    for prog in programs:
        if prog["country"].lower() in prefs:
            for keyword in prog["keywords"]:
                if any(interest in keyword for interest in interests):
                    matches.append(prog)
                    break
    return matches

# Call OpenRouter API with user profile to generate guidance text
def mock_llm_advice(profile, matches):
    messages = [
        {"role": "system", "content": "You are a helpful admission counselor."},
        {"role": "user", "content": f"My interests are {profile['academic_interests']}, strengths are {profile['strengths']}, and preferences are {profile['preferences']}. The matching programs are: {', '.join([p['name'] for p in matches])}. Give me detailed admission advice."}
    ]

    headers = {
        "Authorization": "Bearer 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",
        "HTTP-Referer": "https://chat.openai.com/",
        "X-Title": "Admission-AI"
    }

    payload = {
        "model": "openai/gpt-3.5-turbo",
        "messages": messages
    }

    response = requests.post("https://openrouter.ai/api/v1/chat/completions", headers=headers, json=payload)

    if response.status_code == 200:
        return response.json()['choices'][0]['message']['content']
    else:
        return f"Error from LLM API: {response.status_code} - {response.text}"

# Main controller for the workflow
def main():
    profile = get_student_profile()
    swot = perform_swot(profile)
    matches = match_programs(profile)

    print("\n--- SWOT Analysis ---")
    for k, v in swot.items():
        print(f"{k}: {v}")

    print("\n--- Recommended Programs ---")
    if matches:
        for m in matches:
            print(f"{m['name']} ({m['country']})")
    else:
        print("No programs matched your profile.")

    print("\n--- Personalized Admission Advice ---")
    print(mock_llm_advice(profile, matches))

if __name__ == "__main__":
    main()
