# Yummy Chummy Chatbot

A rule-based food ordering chatbot built in Python with regular expressions and NLTK. It shows the menu, answers price and delivery questions, remembers your name, takes orders with quantities, and gives a running basket summary at checkout.

Built for **CM2015 – Programming With Data** (midterm coursework), BSc Computer Science, University of London.

## Features

- **Intent matching with regex** — patterns and responses live in `intents.json`, so behaviour can change without touching the code.
- **Memory** — remembers the user's name and current order, and fills them into replies (`{name}`, `{order}`).
- **Ordering and basket** — understands phrases like *"I'll take 2 burgers"* or *"can I get a coffee"*, tracks quantities, and adds up the total.
- **Checkout and confirm** — `checkout` shows an itemised summary with the total; `confirm` places the order and empties the basket.
- **Text preprocessing (NLTK)** — tokenising, stopword removal, lemmatization (stemming also supported), and synonym substitution (e.g. *famished → hungry*, *coke → soda*, *chips → fries*).
- **Named entity recognition** — falls back to NLTK NER to pick up names from phrases like *"I am Sarah"*.
- **Sentiment detection** — VADER flags very negative messages and the bot apologises.
- **Keyword-aware fallback** — when nothing matches, it pulls out keywords so the "didn't understand" reply mentions what you said.
- **Graceful degradation** — if an NLTK resource is missing, the bot keeps working with simpler fallbacks.

## Menu

| Item | Price |
|------|-------|
| Pizza | $8 |
| Pasta | $7 |
| Burger | $6 |
| Salad | $5 |
| Fries | $4 |
| Soda / Coffee / Tea | $2 |

## Supported Intents

`greeting` · `goodbye` · `menu` · `hungry` · `price` · `delivery` · `cancel` · `thanks` · `name_response` · `order_response` · `item_inquiry` · `checkout` · `confirm` · `frustrated`

## How It Works

Each message goes through these steps:

1. **Memory check** — `update_memory()` looks for a name or an order and updates the user's state.
2. **Raw intent match** — `match_intent()` tests the input against the regex patterns from `intents.json`.
3. **Preprocessed match** — if nothing matched, the input is cleaned by `ChatbotPreprocessor` (tokenise → remove stopwords → swap synonyms → lemmatize) and matched again.
4. **Sentiment fallback** — if there's still no match and the VADER score is ≤ -0.5, the intent is set to `frustrated`.
5. **Response** — `generate_response()` picks a random reply for the intent and fills in the user's details.

## Project Structure

| File | Purpose |
|------|---------|
| `Yummy Chummy Chatbot - Midterm Project .ipynb` | Notebook with the chatbot code, test cases and a sample run |
| `intents.json` | Menu prices plus each intent's regex patterns and response templates |

### Key Components

| Name | Role |
|------|------|
| `chatbot()` | Main conversation loop |
| `load_files()` | Loads `intents.json` and builds the pattern → intent and intent → response lookups |
| `match_intent()` | Returns the first intent whose pattern matches |
| `update_memory()` | Picks up the user's name, order and quantity |
| `generate_response()` | Picks and fills in a response template |
| `ChatbotPreprocessor` | Tokenising, stopwords, synonyms, stemming and lemmatization |
| `extract_keywords()` | POS-tag-based keyword extraction |
| `extract_entities()` | NLTK named entity recognition |

## Getting Started

### Requirements

- Python 3.10+
- Jupyter Notebook or JupyterLab
- NLTK

### Setup

```bash
git clone https://github.com/wikiahmd/yummy-chummy-chatbot.git
cd yummy-chummy-chatbot
pip install nltk notebook
jupyter notebook
```

The first code cell downloads the NLTK data it needs (`punkt`, `wordnet`, `stopwords`, `vader_lexicon`, `maxent_ne_chunker`, and others).

### Run

1. Open `Yummy Chummy Chatbot - Midterm Project .ipynb`.
2. Choose **Run All**. The helper functions are spread across the notebook, so every cell has to run before the last one starts the bot.
3. Chat in the input box. Type `exit` or `quit` to leave.

## Example Conversation

```
Yummy Chummy: Welcome to the Yummy Chummy Chatbot! Type 'exit' or 'quit' to leave.
You: hi, my name is Wiqar
Yummy Chummy: Hello Wiqar! What can I get you today?
You: what's on the menu?
Yummy Chummy: Today's menu: pizza, burger, fries, pasta, salad, soda, coffee and tea.
You: I'll take 2 pizzas
Yummy Chummy: Great choice! Adding a pizza to your order.
You: can I get a coffee
Yummy Chummy: A coffee it is! Any beverage you'd want with that?
You: checkout
Yummy Chummy: Here's your order, Wiqar:
 2 x pizza - $16
 1 x coffee - $2
 Total: $18
Say 'confirm' to place it!
You: confirm
Yummy Chummy: Order placed, Wiqar! ...
You: exit
Yummy Chummy: Goodbye!
```

## Testing

The **Test Cases** section of the notebook checks:

- **Memory** — names and orders are picked up, and questions like *"do you have pizza?"* are not counted as orders.
- **Intent detection** — each input is marked PASS/FAIL against its expected intent, including synonyms (*"I am absolutely famished"* → `hungry`), gibberish (must not crash), and negative sentiment.

## Customising

To add a new intent, add an entry to `intents.json`:

```json
{
  "tag": "opening_hours",
  "patterns": ["\\b(open|hours|closing)\\b"],
  "responses": ["We're open 10am to 11pm every day, {name}!"]
}
```

Responses can use any key from the user's state: `{name}`, `{order}`, `{qty}`, `{summary}`.

## Known Limitations

- Patterns are checked in the order they appear in the file, so the first matching intent wins.
- `cancel` replies to the user but doesn't remove anything from the basket yet.
- Some reply templates always say "a {order}", even when several were ordered.
- Menu and price replies are fixed text in `intents.json` rather than generated from the `menu` prices.
