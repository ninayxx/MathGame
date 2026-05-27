# Math Quest — Build Plan

A week-by-week plan for building a math game with characters as constants, weapons as formulas, and boss fights as problems. Backend in Python, frontend in HTML/JavaScript, hosted on GitHub + Render.

## Ground rules

- **Everything runs on your laptop** until Week 14. No deployment, no GitHub Pages, no CORS headaches.
- **Commit to GitHub at the end of each week** so you have history and backups.
- **Don't skip ahead.** Each week builds on the last.
- **If a week takes two weeks, that's fine.** The plan is a guide, not a contract.

---

## Architecture overview

Three pieces talking to each other:

- **Frontend** = what runs in the user's browser (HTML, CSS, JavaScript). The page they see, buttons they click.
- **Backend** = a Python program (FastAPI) running on a server. The "brain" — validates answers, decides rewards. Cannot be cheated because users can't see or edit it.
- **Database** = where data is permanently stored (users, scores, problems, unlocks). Only the backend touches it.

The browser never talks to the database directly. It sends requests to the backend, which talks to the database and sends results back.

---

## Week 0 — One-time setup (a single afternoon)

Before week 1, get your tools installed. Do this once, never again.

1. **Install Python 3.11 or newer** from python.org. On the installer, check "Add Python to PATH" — this matters.
2. **Install VS Code** from code.visualstudio.com. Install the "Python" extension (search in the Extensions tab).
3. **Install Git** from git-scm.com. Accept all defaults.
4. **Create a GitHub account** at github.com if you don't have one.
5. **Make a new repo** called `math-quest`. On the GitHub website: New repository → name it → check "Add a README" → Create.
6. **Clone it to your laptop.** On your repo page, click the green "Code" button, copy the URL. Open your terminal, navigate to where you keep projects (`cd Documents`), and run `git clone <paste-url>`. You now have a `math-quest` folder.
7. **Open the folder in VS Code:** File → Open Folder → pick `math-quest`.
8. **Test Python works:** open VS Code's terminal (Terminal → New Terminal) and type `python --version`. You should see something like `Python 3.12.1`.

If anything in this list fails, fix it before moving on. The rest of the plan assumes these are working.

---

## Week 1 — Python refresh + your first SymPy experiments

**Goal:** Get comfortable enough with Python to write the math-checking logic, and meet SymPy.

### Day 1–2: Brush up Python

Confirm these feel comfortable: functions (`def my_function():`), dictionaries (`{"name": "pi", "value": 3.14}`), lists, if/else, for loops. If any feel rusty, skim the first 4 chapters of "Automate the Boring Stuff with Python" (free at automatetheboringstuff.com).

### Day 3: Install SymPy

In VS Code's terminal:
```
pip install sympy
```
Wait ~30 seconds. That's the entire installation.

### Day 4–5: Play with SymPy

Create a file `math_experiments.py` in your repo folder. Type this and run it (right-click → Run Python File):

```python
from sympy import symbols, solve, sympify, sin, cos, pi, sqrt

x = symbols('x')

# Solve a quadratic
print(solve(x**2 + 5*x + 6, x))  # [-3, -2]

# Parse user input from a string
user_input = "-2"
parsed = sympify(user_input)
print(parsed)  # -2

# Check if user's answer matches the solution
solutions = solve(x**2 + 5*x + 6, x)
print(parsed in solutions)  # True

# Some trig
print(sin(pi/2))  # 1
print(sqrt(8))    # 2*sqrt(2)
```

### Day 6–7: Build a real checker

Create `checker.py`:

```python
from sympy import symbols, solve, sympify, simplify

def check_quadratic(a, b, c, user_answer):
    """
    Checks if user_answer is a valid root of ax^2 + bx + c = 0.
    user_answer is a string like "-2" or "1+sqrt(3)"
    """
    x = symbols('x')
    correct_solutions = solve(a*x**2 + b*x + c, x)
    try:
        parsed = sympify(user_answer)
        # Check if user's answer matches any solution (after simplification)
        for sol in correct_solutions:
            if simplify(parsed - sol) == 0:
                return True
        return False
    except:
        return False

# Test it
print(check_quadratic(1, 5, 6, "-2"))    # True
print(check_quadratic(1, 5, 6, "-3"))    # True
print(check_quadratic(1, 5, 6, "5"))     # False
print(check_quadratic(1, 0, -8, "2*sqrt(2)"))  # True
```

### Week 1 deliverable

`checker.py` runs and correctly grades at least 4 different problems. Commit to GitHub:
```
git add .
git commit -m "Week 1: SymPy checker working"
git push
```

---

## Week 2 — Expand the checker, design problem format

**Goal:** A checker that handles the variety of problems your game needs, plus a clean way to represent problems as data.

### Day 1–2: More problem types

Add functions to `checker.py` for:
- Linear equations (`2x + 3 = 7` → `x = 2`)
- Trig values (`sin(π/6) = ?` → `1/2`)
- Simple derivatives if you want calculus

```python
def check_trig(expression_str, user_answer):
    correct = sympify(expression_str)
    parsed = sympify(user_answer)
    return simplify(correct - parsed) == 0

print(check_trig("sin(pi/6)", "1/2"))  # True
```

### Day 3–4: Design a problem format

Every problem in your game needs the same shape. Create `problems.py`:

```python
PROBLEMS = [
    {
        "id": 1,
        "world": "quadratics",
        "difficulty": 1,
        "type": "quadratic",
        "prompt": "Solve: x² + 5x + 6 = 0",
        "data": {"a": 1, "b": 5, "c": 6},
        "currency_cost": 10,
        "reward_currency": 50,
        "reward_theorem": None
    },
    {
        "id": 2,
        "world": "trig",
        "difficulty": 1,
        "type": "trig_value",
        "prompt": "What is sin(π/6)?",
        "data": {"expression": "sin(pi/6)"},
        "currency_cost": 10,
        "reward_currency": 50,
        "reward_theorem": "unit_circle"
    },
]
```

This is the most important design decision of the early weeks. Once your "problem" has a consistent shape, everything else clicks. Write 5–10 problems in this format.

### Day 5–7: Universal checker

One function that takes any problem + user answer and returns correct/incorrect:

```python
def check_answer(problem, user_answer):
    if problem["type"] == "quadratic":
        d = problem["data"]
        return check_quadratic(d["a"], d["b"], d["c"], user_answer)
    elif problem["type"] == "trig_value":
        return check_trig(problem["data"]["expression"], user_answer)
    # ... add more types as needed
    return False
```

### Week 2 deliverable

A test script that loops through all your problems with both correct and wrong answers, printing results. Commit and push.

---

## Week 3 — Your first web server

**Goal:** Wrap your checker in an API so a browser can talk to it.

### Day 1: Install FastAPI

```
pip install fastapi uvicorn
```

### Day 2–3: Hello world server

Create `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Math Quest backend is alive!"}
```

In the terminal:
```
uvicorn main:app --reload
```

Open your browser to `http://localhost:8000`. You should see the JSON message. Open `http://localhost:8000/docs` — FastAPI auto-generates an interactive testing UI. This is gold; you'll use it constantly.

### Day 4–5: First real endpoint

Add to `main.py`:

```python
from pydantic import BaseModel
from problems import PROBLEMS
from checker import check_answer

class Submission(BaseModel):
    problem_id: int
    answer: str
    time_taken: float

@app.get("/api/problems")
def list_problems():
    return PROBLEMS

@app.get("/api/problems/{problem_id}")
def get_problem(problem_id: int):
    for p in PROBLEMS:
        if p["id"] == problem_id:
            return p
    return {"error": "not found"}

@app.post("/api/submit")
def submit(s: Submission):
    problem = next((p for p in PROBLEMS if p["id"] == s.problem_id), None)
    if not problem:
        return {"error": "problem not found"}

    correct = check_answer(problem, s.answer)
    if correct:
        # Faster = better. Award full points if under 10 seconds, scale down.
        speed_bonus = max(0, 100 - int(s.time_taken * 5))
        score = problem["reward_currency"] + speed_bonus
        return {
            "correct": True,
            "score": score,
            "theorem_unlocked": problem["reward_theorem"]
        }
    return {"correct": False, "score": 0}
```

### Day 6–7: Test everything via /docs

Go to `http://localhost:8000/docs`, expand each endpoint, click "Try it out", send test submissions. Verify correct answers return score and wrong ones don't. No frontend yet — the auto-docs page is your test interface.

### Week 3 deliverable

A working backend with three endpoints, fully testable via `/docs`. Commit and push.

---

## Week 4 — The frontend: a real playable page

**Goal:** An HTML page in a browser that lets you solve problems and shows results.

### Day 1: Make the file

Create `index.html` in a new folder `frontend/` inside your repo:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Math Quest</title>
  <style>
    body { font-family: sans-serif; max-width: 600px; margin: 40px auto; padding: 20px; }
    #problem { font-size: 24px; margin: 20px 0; padding: 20px; background: #f0f0f0; border-radius: 8px; }
    #answer { font-size: 18px; padding: 10px; width: 100%; box-sizing: border-box; }
    button { font-size: 18px; padding: 10px 20px; margin-top: 10px; cursor: pointer; }
    #result { margin-top: 20px; font-size: 20px; }
    #timer { font-size: 18px; color: #888; }
    .correct { color: green; }
    .wrong { color: red; }
  </style>
</head>
<body>
  <h1>Math Quest</h1>
  <div id="timer">Time: 0s</div>
  <div id="problem">Loading...</div>
  <input id="answer" placeholder="Your answer (e.g. -2)" />
  <button onclick="submitAnswer()">Submit</button>
  <button onclick="loadProblem()">New Problem</button>
  <div id="result"></div>
  <div>Score: <span id="score">0</span></div>

  <script>
    const BACKEND = 'http://localhost:8000';
    let currentProblem = null;
    let startTime = null;
    let timerInterval = null;
    let totalScore = 0;

    async function loadProblem() {
      const res = await fetch(BACKEND + '/api/problems');
      const problems = await res.json();
      currentProblem = problems[Math.floor(Math.random() * problems.length)];
      document.getElementById('problem').textContent = currentProblem.prompt;
      document.getElementById('answer').value = '';
      document.getElementById('result').textContent = '';
      startTime = Date.now();
      clearInterval(timerInterval);
      timerInterval = setInterval(() => {
        const elapsed = ((Date.now() - startTime) / 1000).toFixed(1);
        document.getElementById('timer').textContent = `Time: ${elapsed}s`;
      }, 100);
    }

    async function submitAnswer() {
      const answer = document.getElementById('answer').value;
      const timeTaken = (Date.now() - startTime) / 1000;
      const res = await fetch(BACKEND + '/api/submit', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({
          problem_id: currentProblem.id,
          answer: answer,
          time_taken: timeTaken
        })
      });
      const data = await res.json();
      const resultDiv = document.getElementById('result');
      if (data.correct) {
        resultDiv.textContent = `Correct! +${data.score} points`;
        resultDiv.className = 'correct';
        totalScore += data.score;
        document.getElementById('score').textContent = totalScore;
      } else {
        resultDiv.textContent = 'Wrong, try again!';
        resultDiv.className = 'wrong';
      }
      clearInterval(timerInterval);
    }

    loadProblem();
  </script>
</body>
</html>
```

### Day 2: Fix CORS

Because your HTML opens from `file://` and your backend is on `localhost`, you'll hit a CORS block. Add to the top of `main.py`:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # fine for local development
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Day 3: Test the full loop

Start your backend (`uvicorn main:app --reload`), then double-click `index.html`. You should see a problem, be able to submit an answer, and see a result. **This is huge — your first end-to-end interaction.**

### Day 4–7: Polish the experience

Add: a "skip/resign" button (costs currency), display the current world, show a list of recent results, make it look a little nicer.

### Week 4 deliverable: THIS IS YOUR MVP

Play it for an hour. Have a friend or family member try it. Take notes on what's confusing or fun. Commit and push.

---

## Week 5 — Database setup (the big jump)

**Goal:** Data survives across sessions. Players have identities. Scores persist.

### Day 1: Install SQLAlchemy

```
pip install sqlalchemy
```

### Day 2–3: Design your schema

Create `database.py`:

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, ForeignKey, DateTime
from sqlalchemy.orm import declarative_base, relationship, sessionmaker
from datetime import datetime

engine = create_engine("sqlite:///math_quest.db")
Base = declarative_base()
SessionLocal = sessionmaker(bind=engine)

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    username = Column(String, unique=True)
    currency = Column(Integer, default=100)
    created_at = Column(DateTime, default=datetime.utcnow)

class Submission(Base):
    __tablename__ = "submissions"
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    problem_id = Column(Integer)
    correct = Column(Integer)  # 1 or 0
    time_taken = Column(Float)
    score = Column(Integer)
    submitted_at = Column(DateTime, default=datetime.utcnow)

class Unlock(Base):
    __tablename__ = "unlocks"
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    theorem_name = Column(String)

Base.metadata.create_all(engine)
```

Run this file once (`python database.py`) — it creates `math_quest.db`, a single file that *is* your database.

### Day 4–5: Wire it into the API

Update `main.py` to create users, save submissions, deduct currency on boss entry, award currency and theorems on success. Add an endpoint to fetch a user's stats.

### Day 6–7: Update the frontend

Create a user on first visit (store user_id in `localStorage`) and send it with every request. Now scores persist when you refresh.

### Week 5 deliverable

Close the browser, reopen, and your stats are still there. Commit and push.

---

## Weeks 6–8 — The game your design describes

**Goal:** Characters, weapons, treasures, boss fights, gating, costs, rewards.

### Week 6: Characters and weapons

Add tables for `Character` (π, e, i, φ) and `Weapon` (sin, cos, quadratic formula). Each character starts with one weapon; players unlock more by completing bosses. Frontend gets a "character select" screen and a "weapons inventory" display.

### Week 7: Boss fights and treasures

A boss fight is a problem with extra constraints: it costs currency to enter, has a time limit, and grants a treasure (theorem) on win. Implement the gating logic: an endpoint that checks whether a user owns the prerequisite weapons before allowing entry. Add a `Theorem` table and the unlock system. Add the "resign" mechanic.

### Week 8: Worlds

Tag every problem with a `world` field (you already did this in Week 2). Frontend gets a world-select screen. Each world has its own list of bosses with their own progression. Build at least 3 worlds with 5+ problems each.

### Weeks 6–8 deliverable

A game with characters, weapons, treasures, bosses, gating, and multiple worlds. Genuinely playable. Commit and push at the end of each week.

---

## Weeks 9–10 — Leaderboards

**Goal:** Show players how they stack up.

### Week 9: Per-problem leaderboards

Add an endpoint `/api/leaderboard/problem/{id}` that returns the top 10 fastest correct solutions for that problem. Show it on each problem's page.

### Week 10: Global leaderboards

Add `/api/leaderboard/global` returning top solvers by total problems solved. Show on a dedicated page.

**The trick with leaderboards:** all scoring happens server-side. The frontend never sends a "score" — it sends an answer and the backend computes the score. Otherwise people will cheat trivially by editing JavaScript.

### Weeks 9–10 deliverable

Working leaderboards. Commit and push.

---

## Weeks 11–13 — User-submitted problems

**Goal:** Players can contribute their own problems.

### Week 11: The submission form

A page where users enter a prompt, a problem type, the problem data (coefficients, expressions), and the correct answer. The form posts to a new endpoint that stores it as a "pending" problem.

### Week 12: Auto-validation

When a problem is submitted, your backend uses SymPy to check that the submitter's answer actually solves the problem they described. If not, reject the submission with an error.

### Week 13: Moderation queue

A simple admin page (just for you) where you review pending submissions and approve them into the main pool. Approved problems start appearing in the right world for everyone.

### Weeks 11–13 deliverable

A working content pipeline where the community can grow the game. Commit and push.

---

## Week 14 — Deploy it to the real internet

**Goal:** Other people can play without being on your laptop.

### Day 1–2: Backend on Render

Sign up at render.com (free tier exists). New → Web Service → connect your GitHub repo. Tell it the start command is `uvicorn main:app --host 0.0.0.0 --port $PORT`. It builds and gives you a URL like `https://math-quest.onrender.com`.

### Day 3: Database upgrade

SQLite works on Render's free tier but loses data on restart. Either accept that, or upgrade to PostgreSQL (Render offers a free postgres database; you'd change one line in `database.py`).

### Day 4–5: Frontend on GitHub Pages

In your repo on github.com: Settings → Pages → Source: deploy from branch → main → `/frontend` folder. Your game is now at `https://your-username.github.io/math-quest/`.

### Day 6: Update the frontend

Point it at your real backend URL. Tighten CORS on the backend to only allow your GitHub Pages URL:

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://your-username.github.io"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Day 7: Test on someone else's device

Send the link to a friend. Watch them play. Note every confusion.

### Week 14 deliverable

A live game on the internet that anyone with the URL can play. Commit and push.

---

## Week 15 onward — Polish, art, and growth

Now the open-ended phase:
- Animations for correct answers
- Sprite art for characters
- Sound effects
- Better visual design
- Real accounts with passwords
- Social features
- Mobile-friendly styling
- More problems, more worlds, more characters

This part has no fixed plan because it depends on what *your* players (and you) want most.

---

## When to design

Design throughout — but the *kind* of design changes:

- **Weeks 1–4:** Sketch on paper or a whiteboard. Ugly is fine. You're answering "what screens do I need and what's on each one?" — not "what color is the button?" Resist the urge to make things pretty here. You'll throw most of it away.
- **Weeks 5–10:** Once the MVP is real and you know what screens exist, start making proper mockups in Figma (free, web-based). Pick an art direction — pixel art? clean vector? hand-drawn? — and commit to one. Polish *some* screens (home, boss fight) but not all.
- **Week 11+:** Full visual treatment. Sprite art, animations, sound effects. If you're not an artist, itch.io has cheap or free asset packs.

The biggest design mistake to avoid: trying to design everything beautifully at the start. You don't yet know what the game *is*. Wait until you've played your own MVP, watched a friend play it, and figured out what's actually fun. *Then* design.

---

## Survival tips

- **Get unstuck fast.** When something doesn't work, search the exact error message on Google. Stack Overflow will have the answer 95% of the time.
- **Commit at the end of every day, not just every week.** Cheap insurance.
- **The hardest weeks are 5 (database) and 14 (deployment).** Expect to spend extra time on those. Everything else is just adding features on top of the foundation.
- **Resist scope creep.** New ideas during weeks 1–4 go in an `IDEAS.md` file, not into the code. Finish the MVP first.
- **"Ship the MVP" means deploy a playable version, even if ugly.** Don't wait for perfect. The opposite — building privately for 6 months — is the trap most beginners fall into.

---

## Project structure (final)

By the end, your repo should look something like:

```
math-quest/
├── README.md
├── PLAN.md                 ← this file
├── IDEAS.md                ← random ideas you have along the way
├── main.py                 ← FastAPI backend entry
├── checker.py              ← SymPy answer validation
├── problems.py             ← problem definitions
├── database.py             ← SQLAlchemy models
├── math_quest.db           ← SQLite file (gitignored)
├── frontend/
│   ├── index.html
│   ├── style.css
│   └── game.js
└── requirements.txt        ← list of pip packages
```
