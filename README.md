# Hagstofa AI – PxWeb API Backend (Track 2)

This project is a chatbot for official statistics from Statistics Iceland (Hagstofa Íslands).
It does not use a pre-built vector database.
It gets the data live from the official PxWeb API.


## 💡 Project Overview & Architecture

### What it Does
You ask a question in English about Icelandic statistics.
The app finds the right table and builds a PX-Web query.
It gets the data and gives a short, clear answer.

### Architecture & Pipeline
1. **Startup (`lifespan`)**: The server reads the metadata of each table once. It keeps the variable codes, labels and allowed values in memory.
2. **Table choice**: The app cleans the question. It removes accents, changes Icelandic letters (þ → th), removes common words (stopwords) and adds synonyms. Then it gives each table a score with weighted keywords. The best table wins. If no table scores high enough, the app says it cannot answer.
3. **Query building (`build_query`)**: The app reads the years in the question. It picks the values that match the question (for example Harpa or Cod). It uses "Total" for the other columns. It asks for 200 values at most (`MAX_CELLS = 200`).
4. **Data retrieval**: The app sends the query to the PX-Web API (`POST`) and gets the values.
5. **Answer**: An LLM writes a short answer from the values. We use `qwen2.5:14b`, run by Ollama. The program does all the calculations (differences, sums, ratios). The LLM only copies the numbers. The program checks the answer: if it has a number that is not in the data, the app shows the raw values instead. The program adds the source at the end.

### Supported Tables
The app can answer questions about 6 official Statistics Iceland tables:
- **Fertility 1853–2025** (`MAN05201`)
- **Households: Tenancy status by type of household 2004–2022** (`LIF03215`)
- **Concerts in concert halls 1999–2019** (`MEN05205`)
- **Catch and value by species and fishing areas 1993–2024** (`SJA09001`)
- **Hospital beds 2007–2020** (`HEI04103`)
- **Museum visits by type of museum and region 1995–2023** (`MEN07002`)



## 🛠️ Requirements

The Python packages are listed in `requirements.txt`:

```
fastapi
uvicorn
requests
pydantic
```

You also need:
1. **Python 3.10+**.
2. An internet connection, to reach the Statistics Iceland PxWeb API (`https://px.hagstofa.is/pxen/api/v1/en/`).
3. **Ollama** with the `qwen2.5:14b` model, to write the answers. It is optional (see Step 2). A GPU is recommended.

## 📂 Repository Structure

The repository contains:

```
Gagnathon2026/
│
├── app.py                      # FastAPI application and PxWeb API calls
├── requirements.txt            # Python packages
├── README.md                   # This file
├── index.html                  # Chat page (frontend)
├── examples.jsonl              # 60 test questions (English and Icelandic) with their expected answers
└── .gitignore                  # Files that Git ignores

```

## ⚙️ Step 1: Environment Setup

Open a terminal in the project folder. Run these commands:

1. **Create a virtual environment:**
   ```
   python -m venv venv
   ```
2. **Activate it:**
   * On Mac / Linux:
     ```
     source venv/bin/activate
     ```
   * On Windows (CMD / PowerShell):
     ```
     venv\Scripts\activate
     ```
3. **Install the packages:**
   ```
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

## 🤖 Step 2: LLM Setup (Ollama)

The LLM turns the values into a short sentence.

1. Install Ollama: https://ollama.com
2. Download the model:
   ```
   ollama pull qwen2.5:14b
   ```
3. Make sure Ollama is running (for example with `ollama serve`).

At startup, the server loads the model into memory. So the first question does not wait.

You can change the settings with environment variables:

| Variable | Default | Meaning |
|---|---|---|
| `OLLAMA_URL` | `http://127.0.0.1:11434` | Address of the Ollama server |
| `OLLAMA_MODEL` | `qwen2.5:14b` | Model that writes the answers |
| `USE_LLM` | `1` | Set to `0` to turn the LLM off |

The app also works without the LLM. It then shows the raw values instead of a sentence.
This also happens if the LLM fails or takes more than 60 seconds.

Example, without the LLM:
```
USE_LLM=0 uvicorn app:app --host 0.0.0.0 --port 8000
```

## 🚀 Step 3: Running the Application

Start the FastAPI server. A Spark Unit (GPU) is recommended: the answers come faster.

```
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

* The server runs at `http://127.0.0.1:8000` (on the server itself).
* Interactive API documentation (Swagger UI): `http://127.0.0.1:8000/docs`
* ReDoc documentation: `http://127.0.0.1:8000/redoc`


## 💻 Step 4: Using the Chatbot

Open this address in a browser:
```
http://<ip_address_of_the_server>:8000
```
It opens a simple chat page. Type a question and read the answer.

* Your computer must be on the same network as the server.
* The chatbot only answers questions about the 6 tables in [Supported Tables](#supported-tables).


## 🧪 Example Prompts & Testing

1. **Museum Visits**: How many museum visits were recorded in 2021?
2. **Hospital Beds**: How many hospital beds were available in 2020?
3. **Fish Catch**: What was the total catch of cod in 2022?
4. **Concerts in Harpa**: How many concerts were held in Harpa in 2017?



## 🛑 `.gitignore`

The `.gitignore` file keeps local files out of Git: the virtual environment and the caches.

```
venv/
__pycache__/
*.pyc
.env
```
