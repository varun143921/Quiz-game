# Quiz-game
import json
import random
import time
import os
from playsound import playsound

QUIZ_FILE = "quiz_data.json"
SCORE_FILE = "scores.txt"
SOUND_FILE = "ding.mp3"  

def load_quiz():
    with open(QUIZ_FILE, "r") as f:
        return json.load(f)

def play_sound():
    try:
        playsound(SOUND_FILE)
    except:
        print("(Sound skipped: ding.mp3 not found)")

def load_leaderboard():
    if not os.path.exists(SCORE_FILE):
        return []
    with open(SCORE_FILE, "r") as f:
        return [line.strip().split(":") for line in f]

def save_score(name, score):
    with open(SCORE_FILE, "a") as f:
        f.write(f"{name}:{score}\n")

def show_leaderboard():
    leaderboard = load_leaderboard()
    if not leaderboard:
        print("No scores yet.")
        return
    sorted_scores = sorted(leaderboard, key=lambda x: int(x[1]), reverse=True)
    print("\n🏆 Top 5 Leaderboard:")
    for i, (user, score) in enumerate(sorted_scores[:5], 1):
        print(f"{i}. {user} - {score} points")

def start_quiz():
    data = load_quiz()
    print(f"\n🎮 {data['quiz_title']}")
    print(f"📅 Created: {data['metadata']['creation_date']}")
    print(f"🧠 Categories: {', '.join(data['metadata']['categories'])}")

    name = input("\nEnter your name: ").strip()
    level = input("Choose difficulty (easy / medium / hard): ").lower()

    questions = []
    for l in data["levels"]:
        if l["difficulty"] == level:
            questions = l["questions"]
            break

    if not questions:
        print("❌ No questions found for that level.")
        return

    score = 0
    random.shuffle(questions)

    for q in questions:
        print(f"\n❓ {q['question']}")
        options = q["options"].copy()
        random.shuffle(options)
        for i, opt in enumerate(options):
            print(f"{chr(65+i)}. {opt}")

        start = time.time()
        answer = input("Your answer (A/B/C/D): ").strip().upper()
        end = time.time()

        if end - start > 15:
            print("⏱️ Time's up! You took too long.")
        else:
            try:
                selected = options[ord(answer) - 65]
                if selected == q["answer"]:
                    print("✅ Correct!")
                    score += 1
                else:
                    print(f"❌ Incorrect. Correct answer: {q['answer']}")
            except:
                print("⚠️ Invalid input. No point awarded.")

        print("📘", q["explanation"])
        play_sound()

    print(f"\n🎯 {name}, your score is: {score}/{len(questions)}")
    save_score(name, score)
    show_leaderboard()

if __name__ == "__main__":
    start_quiz()
