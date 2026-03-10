# LLM Sentiment Evaluation – Flow Diagrams

## 1. Install Required Packages

```
Start
  ↓
Install opik, datasets, dotenv
  ↓
Install OpenAI / Anthropic / Gemini SDKs
  ↓
Install Transformers stack (torch, accelerate, bitsandbytes)
  ↓
Environment Ready
```

---

## 2. Import Core Libraries

```
Start
  ↓
Import Python utilities
(os, json, random, time, gc, typing)
  ↓
Import Data Tools
(pandas, datasets, tqdm)
  ↓
Import Opik Evaluation Utilities
(track, evaluate, metrics base)
  ↓
Import LLM SDKs
(openai, anthropic, google)
  ↓
Import Hugging Face Libraries
(torch, transformers)
  ↓
Print System Info
(torch version, CUDA availability, GPU name)
  ↓
Runtime Ready
```

---

## 3. Configure API Keys + Initialize Opik

```
Start
  ↓
Ask user for secrets
(OPIK_API_KEY, OPIK_WORKSPACE,
OPENAI_API_KEY, HF_TOKEN)
  ↓
Store secrets in environment variables
(os.environ)
  ↓
Create Opik Client
(Opik())
  ↓
Print workspace confirmation
  ↓
Authenticated & Ready
```

---

## 4. Define MODELS Registry

```
Start
  ↓
Create empty MODELS dictionary
  ↓
Add OpenAI model configurations
(provider, model_name, enabled)
  ↓
Add HuggingFace model configurations
(provider, load_in_4bit, tokens, temp)
  ↓
Filter only enabled models
  ↓
Print enabled model details
  ↓
Model Registry Ready
```

---

# 5. HuggingFaceModelManager

## 5A. Initialize Manager

```
Start
  ↓
Create loaded_models cache
  ↓
Detect device
(cuda if available else cpu)
  ↓
Print chosen device
  ↓
Manager Ready
```

---

## 5B. load_model(model_name, load_in_4bit)

```
Start
  ↓
Check cache
  ├─ Model exists → Return cached model
  └─ Model not loaded
        ↓
      Load tokenizer
        ↓
      Ensure pad_token exists
      (fallback to eos_token)
        ↓
      Quantization Decision
        ↓
      If load_in_4bit AND GPU available
          ↓
        Configure BitsAndBytes 4‑bit
        Load quantized model
        dtype = float16

      Else
          ↓
        Load full precision model
        dtype = float32 on CPU
        dtype = float16 on GPU

        ↓
      model.eval()
        ↓
      Store model in cache
        ↓
      Optional GPU memory print
        ↓
      Return model + tokenizer
```

---

## 5C. generate(model_name, prompt)

```
Start
  ↓
Load model + tokenizer
(cache or disk)
  ↓
Tokenize prompt
  ↓
Move tensors to device
  ↓
Run model.generate()
(no_grad)
  ↓
Decode generated tokens
  ↓
Return generated text
```

---

## 5D. clear_cache()

```
Start
  ↓
Delete cached models
  ↓
Run gc.collect()
  ↓
If GPU → torch.cuda.empty_cache()
  ↓
Print memory cleared
  ↓
End
```

---

## 6. Load SST‑2 Dataset + Sample Subset

```
Start
  ↓
Load dataset
load_dataset("glue", "sst2")
  ↓
Select validation split
  ↓
Randomly sample 100 indices
  ↓
Create evaluation subset
validation.select(indices)
  ↓
Print sample size
  ↓
Count label distribution
(negative vs positive)
  ↓
Evaluation Data Ready
```

---

## 7. Define Prompt Template

```
Start
  ↓
Define SENTIMENT_PROMPT
with {text} placeholder
  ↓
Create sample text
  ↓
Format prompt
SENTIMENT_PROMPT.format()
  ↓
Print formatted prompt
  ↓
Prompt Template Verified
```

---

## 8. Convert HF Dataset → Opik Dataset

```
Start
  ↓
Try retrieving dataset
opik_client.get_dataset()

  ├─ Dataset exists
  │      ↓
  │   Return existing dataset

  └─ Dataset not found
        ↓
      Build dataset_items list

      input.text → sentence
      expected_output.label → 0/1
      expected_output.sentiment → positive/negative

        ↓
      Create dataset in Opik
        ↓
      Insert dataset items
        ↓
      Return created dataset
```

---

# 9. Parse Model Output + OpenAI Inference

## 9A. parse_sentiment_response()

```
Start
  ↓
Check empty response
  ├─ Yes → return "negative"
  └─ No

      ↓
Normalize text
(strip + lowercase)

      ↓
Try JSON extraction
(first { to last })

      ├─ Valid JSON
      │      ↓
      │   Extract sentiment
      │   return positive/negative

      └─ JSON parsing failed

      ↓
Keyword detection

"positive" → return positive
"negative" → return negative

      ↓
Fallback
return "negative"
```

---

## 9B. call_openai()

```
Start
  ↓
Create OpenAI client
(using API key)
  ↓
Format sentiment prompt
  ↓
Call chat.completions.create()
  ↓
Extract response text
  ↓
Parse sentiment
(parse_sentiment_response)
  ↓
Return result dictionary

{raw_response,
 predicted_sentiment,
 success,
 tokens}

If error → return failure object
```

---

# 10. HuggingFace Inference + Dispatcher

## 10A. call_huggingface()

```
Start
  ↓
Read model configuration
(name, tokens, temperature, 4bit)
  ↓
Format sentiment prompt
  ↓
Check tokenizer chat template

  ├─ Template exists
  │     ↓
  │  apply_chat_template()

  └─ No template
        ↓
      Use raw prompt

  ↓
Generate response locally
hf_manager.generate()
  ↓
Parse predicted sentiment
  ↓
Return result dictionary

{raw_response,
 predicted_sentiment,
 success}

If error → return failure
```

---

## 10B. call_llm()

```
Start
  ↓
provider = model_config["provider"]

  ├─ openai
  │     ↓
  │  call_openai()

  ├─ huggingface
  │     ↓
  │  call_huggingface()

  └─ unknown
        ↓
      Return provider error

End
```

---

## 11. Quick Sanity Test (All Models)

```
Start
  ↓
Define test_text
  ↓
Loop through MODELS

  ↓
Skip disabled models
  ↓
Call call_llm()

  ├─ Success
  │     ↓
  │ Print prediction
  │ Print raw snippet
  │ Print token usage

  └─ Failure
        ↓
      Print error

Continue next model

End (sanity test complete)
```

---

