# 🍳 AI Recipe Chef

An intelligent, interactive cooking assistant built with **Qwen3**, **Ollama**, **LangGraph**, and **Gradio**.

The AI Recipe Chef acts as an agentic cooking assistant that can generate recipes dynamically, guide users through recipes step-by-step, answer cooking questions, scale ingredients, suggest ingredient substitutions, and manage cooking timers through a conversational interface.

---

## 🚀 Features

### 🤖 AI Recipe Generation

Generate complete recipes from natural-language requests.

Example:

```text
I want chicken biryani for 5 people
```

The agent generates:

- Recipe name
- Serving size
- Ingredients with quantities
- Sequential cooking instructions

---

### 🧠 Intent Detection

The system uses Qwen3 to identify the user's intent and route the request to the appropriate agent node.

Supported intents:

```text
START
NEXT_STEP
QUESTION
REPEAT
SCALE
SUBSTITUTE
TIMER
OTHER
```

Example:

```text
"Start cooking"
        ↓
START
        ↓
Recipe Generator
```

---

### 👨‍🍳 Step-by-Step Cooking Assistant

The assistant guides the user through the recipe one step at a time.

Example:

```text
User: Done

AI:
Step 2:
Add the chopped onions and cook until golden brown.
```

The system maintains:

- Current cooking step
- Completed steps
- Recipe progress

---

### 🔄 Repeat Current Step

Users can ask:

```text
Repeat the current step
```

The assistant repeats the current cooking instruction without advancing the recipe.

---

### ❓ Cooking Questions

Users can ask questions while cooking without losing their current position.

Examples:

```text
Can I cook this on low heat?

How do I know when the onions are ready?

Can I use a pressure cooker?
```

---

### ⚖️ Ingredient Scaling

The recipe can be dynamically scaled.

Example:

```text
I want chicken curry for 4 people
```

Then:

```text
Actually, make it for 8 people
```

The assistant recalculates ingredient quantities while keeping the same recipe.

---

### 🔄 Ingredient Substitution

Users can replace ingredients during cooking.

Example:

```text
I don't have chicken, can I use paneer instead?
```

The agent updates the recipe ingredients and, when necessary, adjusts the cooking instructions.

Other examples:

```text
Can I replace butter with oil?

I don't have garam masala.

Can I use coconut milk instead?
```

---

### ⏱️ Cooking Timer

The assistant understands timer requests such as:

```text
Set a timer for 10 minutes
```

or:

```text
Set a timer for 1 hour
```

The system stores:

- Timer duration
- Timer start time
- Remaining time

The Gradio interface provides a live countdown.

Example:

```text
⏱️ 09:42 remaining
```

When the timer finishes:

```text
🔔 Timer finished!
```

---

## 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │       Gradio        │
                    │    Chat Interface   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    LangGraph Agent  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Intent Detection  │
                    │       Qwen3         │
                    └──────────┬──────────┘
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
             ▼                 ▼                 ▼
       START / Recipe       Cooking          Information
        Generation           Flow              Requests
             │                 │                 │
             ▼                 ▼                 ▼
       Generate Recipe     Next Step        Questions
             │             Repeat Step
             │
             ├───────────────┐
             │               │
             ▼               ▼
          Scaling       Substitution
             │               │
             └───────┬───────┘
                     ▼
                  Timer
                     │
                     ▼
              Live Countdown
```

---

## 🧩 LangGraph Workflow

The current graph contains the following nodes:

```text
detect_intent
       │
       ├── START ──────────────→ generate_recipe
       │                              │
       │                              ▼
       │                         start_recipe
       │
       ├── NEXT_STEP ──────────→ advance_step
       │                              │
       │                              ▼
       │                       cooking_response
       │
       ├── QUESTION ───────────→ answer_question
       │
       ├── REPEAT ─────────────→ repeat_step
       │
       ├── SCALE ──────────────→ scale_recipe
       │
       ├── SUBSTITUTE ─────────→ substitute_ingredient
       │
       └── TIMER ──────────────→ timer_agent
```

---

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core programming language |
| Qwen3:8b | Local language model |
| Ollama | Local LLM runtime |
| LangChain | LLM integration |
| LangGraph | Agent orchestration |
| Gradio | Interactive web interface |
| Google Colab | Development environment |
| PyTorch / CUDA | GPU acceleration through Ollama environment |

---

## 💻 Environment

The project is currently designed to run in **Google Colab**.

Recommended environment:

```text
Google Colab
Python 3.x
GPU Runtime
```

Qwen3 is executed through Ollama.

---

# ⚙️ Installation

## 1. Install Ollama

Run in Google Colab:

```python
!apt-get update -qq
!apt-get install -y -qq zstd
!curl -fsSL https://ollama.com/install.sh | sh
```

---

## 2. Start Ollama

Use the following reliable startup function:

```python
import subprocess
import time
import requests

OLLAMA_URL = "http://127.0.0.1:11434"

def ensure_ollama():

    try:
        response = requests.get(
            f"{OLLAMA_URL}/api/tags",
            timeout=3
        )

        if response.status_code == 200:
            print("✅ Ollama is already running")
            return

    except Exception:
        pass

    print("🚀 Starting Ollama...")

    subprocess.Popen(
        ["ollama", "serve"],
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL
    )

    for i in range(20):

        time.sleep(1)

        try:
            response = requests.get(
                f"{OLLAMA_URL}/api/tags",
                timeout=2
            )

            if response.status_code == 200:
                print("✅ Ollama started successfully")
                return

        except Exception:
            pass

    raise RuntimeError("❌ Ollama failed to start")


ensure_ollama()
```

---

## 3. Install Qwen3

```python
!ollama pull qwen3:8b
```

Verify:

```python
!ollama list
```

You should see:

```text
qwen3:8b
```

---

## 4. Install Python Dependencies

```python
!pip install -q -U \
    langgraph \
    langchain \
    langchain-core \
    langchain-ollama \
    gradio
```

---

# 🧠 Configure Qwen3

Create the LangChain Ollama model:

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="qwen3:8b",
    temperature=0.2,
    base_url="http://127.0.0.1:11434"
)
```

Test the connection:

```python
response = llm.invoke(
    "Reply with exactly: Ollama is working"
)

print(response.content)
```

Expected:

```text
Ollama is working
```

---

# 📦 Recipe State

The LangGraph state contains the recipe and cooking progress:

```python
from typing import TypedDict, List
from langchain_core.messages import BaseMessage

class RecipeState(TypedDict):
    messages: List[BaseMessage]

    recipe_name: str
    servings: int

    ingredients: List[str]
    steps: List[str]

    current_step: int
    completed_steps: List[int]

    intent: str

    timer_seconds: int
    timer_started_at: float
```

---

# 🍲 Example Recipe Generation

The agent can receive:

```text
I want chicken biryani for 5 people
```

and dynamically generate a structured recipe:

```json
{
    "recipe_name": "Chicken Biryani",
    "servings": 5,
    "ingredients": [
        "500g chicken",
        "2 cups basmati rice",
        "2 onions",
        "3 tomatoes"
    ],
    "steps": [
        "Marinate the chicken.",
        "Cook the onions.",
        "Add the chicken and spices.",
        "Add the rice.",
        "Cook until the rice is tender."
    ]
}
```

---

# 🔀 Intent Routing

The intent detector classifies the user's request.

Example:

```python
START
```

routes to:

```text
generate_recipe
```

while:

```python
NEXT_STEP
```

routes to:

```text
advance_step
```

The routing function is:

```python
def route_intent(state: RecipeState):

    intent = state["intent"]

    if intent == "NEXT_STEP":
        return "advance"

    elif intent == "QUESTION":
        return "question"

    elif intent == "REPEAT":
        return "repeat"

    elif intent == "START":
        return "start"

    elif intent == "SCALE":
        return "scale"

    elif intent == "SUBSTITUTE":
        return "substitute"

    elif intent == "TIMER":
        return "timer"

    else:
        return "question"
```

---

# 💬 Gradio Interface

The application provides:

- Chat interface
- Start Cooking button
- Done button
- Repeat Step button
- Timer display
- Live countdown

Example:

```text
┌─────────────────────────────────────────────┐
│              🍳 AI Recipe Chef              │
├─────────────────────────────────────────────┤
│                                             │
│ AI: Step 3: Add the chicken and spices.    │
│                                             │
│ User: Done                                  │
│                                             │
├─────────────────────────────────────────────┤
│ ⏱️ 08:42 remaining                          │
├─────────────────────────────────────────────┤
│ [ Enter your message... ]        [ Send ]   │
│                                             │
│ [🍳 Start] [✅ Done] [🔄 Repeat Step]      │
└─────────────────────────────────────────────┘
```

---

# 🧪 Example Conversation

### Start a recipe

```text
User:
I want chicken curry for 4 people
```

```text
AI:
🍳 Chicken Curry

Servings: 4

Step 1:
Heat the oil in a large pan over medium heat.
```

### Continue

```text
User:
Done
```

```text
AI:
Step 2:
Add the onions and cook until golden brown.
```

### Repeat

```text
User:
Repeat the current step
```

```text
AI:
Step 2:
Add the onions and cook until golden brown.
```

### Ask a question

```text
User:
Can I cook the onions on low heat?
```

```text
AI:
Yes. Cook them slowly over low to medium heat...
```

### Scale

```text
User:
Make this for 8 people
```

```text
AI:
The recipe has been scaled to 8 servings.
```

### Substitute

```text
User:
I don't have chicken. Can I use paneer?
```

```text
AI:
Yes. Paneer can be used instead of chicken. I've updated the recipe.
```

### Timer

```text
User:
Set a timer for 10 minutes
```

```text
AI:
⏱️ Timer started for 10 minutes.
```

The interface then displays:

```text
⏱️ 09:59 remaining
```

---

# ⚠️ Common Issue: Ollama Connection Refused

If you see:

```text
httpx.ConnectError: [Errno 111] Connection refused
```

the Ollama server has stopped.

Run:

```python
ensure_ollama()
```

Then verify:

```python
!ollama list
```

Then test:

```python
llm.invoke("Hello")
```

Only after that should you run the LangGraph agent.

---

# 🔄 Reset Conversation

For testing a new recipe:

```python
conversation_state = {
    "messages": [],
    "recipe_name": "",
    "servings": 0,
    "ingredients": [],
    "steps": [],
    "current_step": 0,
    "completed_steps": [],
    "intent": "",
    "timer_seconds": 0,
    "timer_started_at": 0.0
}
```

---

# 🔐 Current Architecture Limitation

The current prototype uses a global:

```python
conversation_state
```

This is suitable for:

- Development
- Testing
- Single-user Colab sessions

It is **not yet suitable for multiple simultaneous users**.

For production, the state should be associated with a unique user/session/thread.

---

# 🚧 Future Improvements

Planned improvements include:

- [ ] Multi-user session management
- [ ] LangGraph checkpointer
- [ ] Persistent recipe history
- [ ] Voice input
- [ ] Voice output
- [ ] Speech-to-text
- [ ] Image-based ingredient recognition
- [ ] Pantry management
- [ ] Shopping list generation
- [ ] Nutrition information
- [ ] Dietary preference handling
- [ ] Allergy detection
- [ ] Step-specific timers
- [ ] Multiple simultaneous timers
- [ ] Timer notifications
- [ ] Recipe recommendation system
- [ ] Recipe search and retrieval
- [ ] Food image understanding
- [ ] Mobile application
- [ ] Backend API with FastAPI
- [ ] Database integration
- [ ] Production deployment

---

# 🔮 Future Agent Architecture

The long-term architecture can evolve into a multi-agent cooking assistant:

```text
                         ┌───────────────────┐
                         │   User / Voice    │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │   Orchestrator    │
                         │    LangGraph      │
                         └─────────┬─────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
          ▼                        ▼                        ▼
   Recipe Agent              Cooking Agent            Timer Agent
          │                        │                        │
          ▼                        ▼                        ▼
  Recipe Generation          Step Tracking           Countdown
  Recipe Scaling             Questions               Notifications
  Substitution               Progress
          │
          ▼
   ┌─────────────────────────────────────────┐
   │             Knowledge Layer             │
   │ Recipes • Ingredients • Nutrition       │
   │ Cooking Techniques • User Preferences   │
   └─────────────────────────────────────────┘
```

---

# 📁 Suggested Project Structure

As the project grows, organize it as:

```text
ai-recipe-chef/
│
├── app/
│   ├── graph.py
│   ├── state.py
│   ├── intents.py
│   │
│   ├── agents/
│   │   ├── recipe_agent.py
│   │   ├── cooking_agent.py
│   │   ├── substitution_agent.py
│   │   ├── scaling_agent.py
│   │   └── timer_agent.py
│   │
│   └── ui/
│       └── gradio_app.py
│
├── requirements.txt
├── README.md
└── .gitignore
```

---

# 🎯 Project Goal

The goal of **AI Recipe Chef** is to create an intelligent conversational cooking assistant that can understand natural-language cooking requests and actively assist users throughout the entire cooking process.

Instead of simply generating a recipe, the system maintains cooking context and allows the user to interact naturally:

```text
Generate → Cook → Ask → Modify → Scale → Substitute → Time → Complete
```

---

## 👨‍💻 Author

**Basil Sajeev**

MSc Computer Science | AI/ML Engineer

---

## ⭐ Project Status

**Current Status: Prototype / Active Development**

Core capabilities currently implemented:

- ✅ Local Qwen3 LLM
- ✅ Ollama
- ✅ LangGraph agent
- ✅ Dynamic recipe generation
- ✅ Intent detection
- ✅ Step-by-step cooking
- ✅ Recipe scaling
- ✅ Ingredient substitution
- ✅ Cooking timer
- ✅ Gradio interface
- ✅ Live timer display

The project is being developed toward a more advanced **agentic, multimodal cooking assistant**.
