# Assignment: Build a Text Summarizer with Granite & Ollama

## 🎯 Learning Objectives

By completing this assignment, you will:
- Understand how to serve local LLMs with Ollama
- Practice prompt engineering for a real use case (summarization)
- Build a working application that interacts with an LLM via REST API
- Learn to iterate and improve prompts based on results
- Handle errors gracefully in production-like code

---

## 📋 Assignment Overview

**Build an interactive app that summarizes text documents using the Granite model.**

Your app should:
1. Accept text input from the user
2. Send a well-crafted summarization prompt to Granite via Ollama
3. Display the summary to the user
4. Allow the user to refine the summary (optional: control summary length)

---

## ⚙️ Setup & Verification

### Step 1: Verify Ollama is Running

```bash
curl localhost:11434
```

Expected output:
```
Ollama is running
```

If you see an error, start Ollama first:
```bash
ollama serve
```

### Step 2: Verify Granite Model is Downloaded

```bash
ollama list
```

You should see `granite3.1-dense:2b` in the list. If not, download it:
```bash
ollama pull granite3.1-dense:2b
```

### Step 3: Test the API

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "granite3.1-dense:2b",
  "prompt": "What is 2 + 2?",
  "stream": false
}'
```

Expected: A JSON response with the model's answer in the `"response"` field.

---

## 🔨 Building Your App

### Phase 1: Basic Implementation (Required)

Create a script that:

1. **Takes text input** - Either from a file or command line
2. **Crafts a summarization prompt** - Use prompt engineering!
3. **Calls the Ollama API** - Send request to `http://localhost:11434/api/generate`
4. **Displays the output** - Print the summary to the user

**Skeleton code (Python):**

```python
import requests
import json

def summarize_text(text, summary_length="medium"):
    """
    Summarize the given text using Granite model.

    Args:
        text: The text to summarize
        summary_length: "short" (1-2 sentences), "medium" (3-5 sentences), "long" (detailed)

    Returns:
        The summary as a string
    """

    # TODO: Create a well-engineered prompt for summarization
    # Hint: Include instructions on summary length, tone, and format
    prompt = f"""You are a helpful assistant that creates clear, concise summaries.

Summarize the following text in {summary_length} form.
Be accurate and capture the main points.

Text to summarize:
{text}

Summary:"""

    # TODO: Make API call to Ollama
    # Hint: Handle connection errors gracefully
    try:
        response = requests.post(
            'http://localhost:11434/api/generate',
            json={
                'model': 'granite3.1-dense:2b',
                'prompt': prompt,
                'stream': False
            },
            timeout=60
        )
        response.raise_for_status()
    except requests.exceptions.ConnectionError:
        return "Error: Could not connect to Ollama. Is it running on localhost:11434?"
    except requests.exceptions.Timeout:
        return "Error: Request timed out. The model is taking too long."
    except Exception as e:
        return f"Error: {str(e)}"

    # TODO: Extract and return the response
    result = response.json()
    return result.get('response', 'No response from model')


if __name__ == "__main__":
    # Test with a sample document
    sample_text = """
    Machine learning is a subset of artificial intelligence that focuses on
    enabling computers to learn from data without being explicitly programmed.
    Instead of following pre-defined rules, ML systems identify patterns in data
    and improve their performance through experience. Common applications include
    image recognition, natural language processing, and recommendation systems.
    The field has grown exponentially due to increased computing power and
    availability of large datasets.
    """

    print("Original text:")
    print(sample_text)
    print("\n" + "="*50 + "\n")

    summary = summarize_text(sample_text, summary_length="short")
    print("Summary:")
    print(summary)
```

---

### Phase 2: Prompt Engineering & Iteration (Required)

**Your prompt is critical!** Spend time refining it.

#### Task: Test Different Prompts

Run at least 3 different prompts and compare:

**Prompt 1 - Minimal:**
```
Summarize: {text}
```

**Prompt 2 - Structured:**
```
You are a summarization expert.
Summarize the text below in exactly 2-3 sentences.
Focus on the main ideas and key facts.

Text: {text}

Summary:
```

**Prompt 3 - Your Own:**
```
(Design your own - be creative!)
```

📝 **Document your findings:**
- Which prompt worked best?
- Why do you think it worked better?
- What did you learn about prompt engineering?

---

### Phase 3: User Interaction (Recommended)

Enhance your app to accept user input:

```python
def interactive_summarizer():
    """Allow user to input text and get summaries."""

    while True:
        print("\n" + "="*50)
        print("Text Summarizer with Granite")
        print("="*50)

        # Get user input
        user_input = input("Paste your text (or 'quit' to exit):\n> ")
        if user_input.lower() == 'quit':
            break

        # Get summary length preference
        length = input("Summary length - short/medium/long? (default: medium) > ").lower()
        if length not in ["short", "medium", "long"]:
            length = "medium"

        # Generate summary
        print("\nGenerating summary...")
        summary = summarize_text(user_input, summary_length=length)
        print(f"\nSummary:\n{summary}")


if __name__ == "__main__":
    interactive_summarizer()
```

---

## ✅ Acceptance Criteria

Your submission should include:

- [ ] **Working app** - Runs without crashing and produces summaries
- [ ] **Error handling** - Gracefully handles missing Ollama, timeouts, invalid input
- [ ] **Prompt engineering documentation** - Show at least 2 different prompts and explain which works better
- [ ] **Test cases** - Include 2-3 sample texts and their outputs
- [ ] **Code comments** - Explain the key parts of your implementation
- [ ] **(Optional) Enhancements** - Any extra features (multi-language, keyword extraction, etc.)

---

## 🧪 Testing Your App

### Test Case 1: Basic Functionality

**Input:**
```
The Python programming language was created by Guido van Rossum in 1991.
It emphasizes code readability and simplicity, making it popular for beginners
and experts alike. Python is used in web development, data science, AI, and
automation. Its large standard library and active community contribute to its success.
```

**Expected output:**
A 2-3 sentence summary about Python

---

### Test Case 2: Error Handling

**What to test:**
- Stop Ollama and try to run your app → Should show a helpful error message
- Send very long text → Should still work or warn gracefully
- Send empty text → Should handle gracefully

---

### Test Case 3: Prompt Comparison

Create a document showing:
1. Original text
2. Summary from Prompt 1 (minimal)
3. Summary from Prompt 2 (structured)
4. Which is better and why?

---

## 🎮 Extra Challenges (Optional)

Pick one or more to enhance your grade:

### Challenge 1: Keyword Extraction
Modify your app to also extract 3-5 key keywords from the text.

```python
def extract_keywords(text):
    prompt = f"""Extract 5 important keywords from this text.
Return them as a comma-separated list.

Text: {text}

Keywords:"""
    # ... call API and return keywords
```

### Challenge 2: Adjustable Summary Length
Allow users to specify exact number of sentences:

```python
"Summarize in exactly {num_sentences} sentences."
```

### Challenge 3: Multi-Language Support
Test if your summarizer works with non-English text. Document findings.

### Challenge 4: Batch Processing
Modify your app to process multiple documents from a folder:

```python
import os

def summarize_folder(folder_path):
    """Summarize all .txt files in a folder."""
    for filename in os.listdir(folder_path):
        if filename.endswith('.txt'):
            with open(os.path.join(folder_path, filename)) as f:
                text = f.read()
            summary = summarize_text(text)
            print(f"{filename}: {summary}\n")
```

### Challenge 5: A/B Testing
Implement two completely different prompting strategies and compare them:

```python
def summarize_with_strategy_a(text):
    # Strategy: Step-by-step reasoning
    prompt = "First identify the main topic, then list key points, then write summary..."

def summarize_with_strategy_b(text):
    # Strategy: Direct minimal instructions
    prompt = "Summarize in 3 sentences..."

# Compare outputs and measure which is "better"
```

---

## 📚 Reference: Prompt Engineering Tips for This Task

### ✅ Good Summarization Prompts

```
You are an expert summarization assistant.
Create a concise summary of the following text.
Capture the main ideas and important details.
Use simple, clear language.

Text: {text}

Summary (2-3 sentences):
```

```
Summarize the following text in {num_sentences} sentences.
Focus on WHO did WHAT and WHY.
Avoid unnecessary details.

Text: {text}

Summary:
```

### ❌ Weak Prompts (Don't Use)

```
Summarize this
```

```
Make it shorter
```

```
What is this about?
```

---

## 📦 Submission Checklist

- [ ] Code file(s) - Python script or notebook
- [ ] README - How to run your app
- [ ] Prompt comparison document - Show your prompt engineering work
- [ ] Test results - Sample inputs and outputs
- [ ] Any extra features documented

---

## 🤔 Reflection Questions (Optional but Recommended)

1. How did changing your prompt affect the quality of summaries?
2. What challenges did you face when working with the API?
3. How would you improve this app further?
4. What other tasks could you use Ollama for?
5. How is this different from using a cloud API like OpenAI?

---

## 🆘 Troubleshooting

| Problem | Solution |
|---------|----------|
| "Connection refused" | Make sure Ollama is running (`ollama serve`) |
| "Model not found" | Download Granite (`ollama pull granite3.1-dense:2b`) |
| "Timeout error" | Model is slow on first run, try again or increase timeout |
| "Weird responses" | Your prompt might be unclear - try rephrasing it |
| "Out of memory" | Close other applications or use `granite3.1-dense:2b` (smallest) |

---

## 📖 Resources

- Ollama Docs: https://ollama.ai
- Granite Model: https://www.ibm.com/granite
- Prompt Engineering Guide: https://www.ibm.com/think/topics/prompt-engineering
- Python Requests Library: https://docs.python-requests.org/

---

## 🎯 Learning Path

After completing this assignment:
1. Try building a different app (chatbot, code reviewer, translator)
2. Experiment with different Ollama models
3. Combine with LangChain for more advanced features
4. Deploy your app to share with others
