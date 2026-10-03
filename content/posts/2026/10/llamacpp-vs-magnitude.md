---
title: "Testing Magnitude vs llama.cpp: Is It Really 2x Faster?"
date: 2026-10-03T13:00:00+08:00
categories:
- tech
tags:
- llamapp.cpp
- magnitude
- dflash
---

Several days ago, I came across a claim from Magnitude stating that it's up to 2x faster than llama.cpp, and—more interestingly for me—that it supports AMD GPUs as well.

Naturally, I was skeptical. A 2x speedup over llama.cpp is a bold claim, especially on AMD hardware where the ROCm ecosystem can be hit-or-miss. So today, I decided to give it a try and run my own benchmarks.


## Setup

I used the following build for Magnitude:


- llama-b11368-bin-ubuntu-rocm-10.0-x64.tar.gz
- magnitude-desktop_0.2.4-54_amd64.deb

The test model was Qwen3.6-35B-A3B:

- Main model: unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL
- Draft model: Qwen3.6-35B-A3B-DFlash-Q8_0.gguf

The idea was to test both engines with and without speculative decoding (the DFlash draft model), to see where the performance gap actually shows up.



## Results


|Engine| Model | Tokens/s|
| ---  | ---  | ---              |
|llama-server | unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL | 42|
|llama-server | unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL + Qwen3.6-35B-A3B-DFlash-Q8_0.gguf | 84|
|magnitudedev | unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL + Qwen3.6-35B-A3B-DFlash-Q8_0.gguf | 78|


###  llama-server | Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL

```bash
llama-server \
  -hf unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL \
  --ctx-size 262144 \
  --flash-attn on \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --batch-size 2048 \
  --ubatch-size 1024 \
  --n-gpu-layers 999 \
  --jinja \
  --temp 0.6 \
  --top-k 20 \
  --top-p 0.95 \
  --min-p 0.05 \
  --repeat-penalty 1.0
curl http://localhost:8080/v1/chat/completions   \
  -H "Content-Type: application/json"    \
   -d '{"messages": [{"role": "user", "content": "Hello!"}]}'
{"choices":[{"finish_reason":"stop","index":0,"message":{"role":"assistant","content":"Hello! How can I help you today? 😊","reasoning_content":"Here's a thinking process:\n\n1.  **Analyze User Input:**\n   - User said: \"Hello!\"\n   - This is a simple greeting, no specific question or request attached.\n\n2.  **Identify Intent:**\n   - The user is initiating a conversation.\n   - Expected response: A friendly greeting back, possibly asking how I can help.\n\n3.  **Determine Response Strategy:**\n   - Acknowledge the greeting warmly.\n   - Offer assistance.\n   - Keep it concise and open-ended.\n\n4.  **Draft Response (Mental):**\n   Hello! How can I assist you today?\n\n5.  **Refine Response:**\n   - Matches tone: Friendly, professional.\n   - Matches intent: Opens the door for the user to ask a question or make a request.\n   - No extra fluff.\n\n6.  **Final Output Generation:** (matches the refined draft)\n   \"Hello! How can I help you today?\"✅\n"}}],"created":1791011322,"model":"unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL","system_fingerprint":"b11368-1fb7ef3e3","object":"chat.completion","usage":{"completion_tokens":226,"prompt_tokens":12,"total_tokens":238,"prompt_tokens_details":{"cached_tokens":0}},"id":"chatcmpl-6bxgsDyw75MLnWrOgac0YPFZOqDdQEYV","timings":{"cache_n":0,"prompt_n":12,"prompt_ms":320.421,"prompt_per_token_ms":26.70175,"prompt_per_second":37.45072888481092,"predicted_n":226,"predicted_ms":5333.842,"predicted_per_token_ms":23.705964444444444,"predicted_per_second":42.18347675090489}}
```

### llama-server | unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL + Qwen3.6-35B-A3B-DFlash-Q8_0.gguf

image::/images/Qwen3.6_MTP_magnitude.png[unsloth/Qwen3.6-35B-A3B-MTP-GGUF]

image::/images/Qwen3.6_magnitude_dflash[Qwen3.6-35B-A3B-DFlash-Q8_0]


```bash
hf download unsloth/Qwen3.6-35B-A3B-MTP-GGUF Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf --local-dir ~/models
hf download magnitudedev/Qwen3.6-35B-A3B-DFlash-GGUF Qwen3.6-35B-A3B-DFlash-Q8_0.gguf --local-dir ~/models
llama-server \
  -m  ~/models/Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf \
  -md ~/models/Qwen3.6-35B-A3B-DFlash-Q8_0.gguf \
  --spec-type draft-dflash \
  --spec-draft-n-max 4 \
  -ngl 999 -fa on -c 262144


curl http://localhost:8080/v1/chat/completions   \
  -H "Content-Type: application/json"    \
   -d '{"messages": [{"role": "user", "content": "Hello!"}]}'
{"choices":[{"finish_reason":"stop","index":0,"message":{"role":"assistant","content":"Hello! How can I assist you today? 😊","reasoning_content":"Here's a thinking process:\n\n1.  **Analyze User Input:**\n   - User said: \"Hello!\"\n   - This is a simple greeting, no specific question or request attached.\n\n2.  **Identify Intent:**\n   - The user is initiating a conversation.\n   - Expected response: A friendly greeting back, possibly asking how I can help.\n\n3.  **Determine Response Strategy:**\n   - Acknowledge the greeting warmly.\n   - Offer assistance.\n   - Keep it concise and open-ended.\n   - Match the tone (friendly, professional).\n\n4.  **Draft Response (Mental):**\n   - \"Hello! How can I assist you today?\"\n   - Alternative: \"Hi there! What can I help you with?\"\n   - Both are appropriate. I'll go with a clear, friendly version.\n\n5.  **Refine Response:**\n   - \"Hello! 👋 How can I help you today?\" (Adding a subtle emoji for warmth, though optional. I'll keep it text-only for professionalism unless the context suggests otherwise. Actually, standard AI practice is to keep it clean and professional.)\n   - Final: \"Hello! How can I assist you today?\"\n\n6.  **Self-Correction/Verification:**\n   - Does it match the user's tone? Yes.\n   - Is it open-ended? Yes.\n   - Does it invite further interaction? Yes.\n   - No overcomplication needed.\n\n   All good. Proceed.✅\n"}}],"created":1791021548,"model":"/home/jackl/models/Qwen3.6-35B-A3B-UD-Q8_K_XL.gguf","system_fingerprint":"b11368-1fb7ef3e3","object":"chat.completion","usage":{"completion_tokens":336,"prompt_tokens":12,"total_tokens":348,"prompt_tokens_details":{"cached_tokens":0}},"id":"chatcmpl-ltl9YEXZFMZMi9tdfzxHvWI6yDgi0n7A","timings":{"cache_n":0,"prompt_n":12,"prompt_ms":246.589,"prompt_per_token_ms":20.549083333333332,"prompt_per_second":48.663971223371675,"predicted_n":336,"predicted_ms":3967.447,"predicted_per_token_ms":11.84312537313433,"predicted_per_second":84.4371708053063,"draft_n":312,"draft_n_accepted":257}}
```

### magnitudedev | unsloth/Qwen3.6-35B-A3B-MTP-GGUF:UD-Q8_K_XL + Qwen3.6-35B-A3B-DFlash-Q8_0.gguf

```bash
# 
curl http://127.0.0.1:10100/inference/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen3.6-35b-a3b:gguf:q8","messages":[{"role":"user","content":"Hello"}]}'
{"id":"chatcmpl-icn-1","object":"chat.completion","created":1791006357,"model":"qwen3.6-35b-a3b:gguf:q8","choices":[{"index":0,"message":{"role":"assistant","content":"Hello! How can I assist you today? Feel free to ask me anything or let me know what you'd like help with. 😊","reasoning_content":"Here's a thinking process:\n\n1.  **Analyze User Input:**\n   - User said: \"Hello\"\n   - This is a standard greeting, no specific question or request attached.\n\n2.  **Identify Intent:**\n   - The user is initiating a conversation.\n   - Expected response: A friendly greeting, acknowledgment, and an invitation to ask a question or state their needs.\n\n3.  **Determine Response Strategy:**\n   - Keep it warm and professional.\n   - Acknowledge the greeting.\n   - Offer assistance.\n   - Keep it concise.\n\n4.  **Draft Response (Mental):**\n   Hello! How can I assist you today? Feel free to ask me anything or let me know what you'd like help with.\n\n5.  **Refine Response:**\n   - Check tone: Friendly, helpful, open-ended.\n   - Check length: Concise.\n   - Matches expected AI behavior: Yes.\n\n   Final version: \"Hello! How can I assist you today? Feel free to ask me anything or let me know what you'd like help with.\"\n\n6.  **Output Generation:** (Proceeds to output)✅\n"},"finish_reason":"stop"}],"usage":{"prompt_tokens":11,"completion_tokens":283,"total_tokens":294,"prompt_tokens_details":{"cached_tokens":0}},"timings":{"cache_n":0,"prompt_n":11,"prompt_ms":129.006136,"time_to_first_token_ms":374.97332700000004,"prompt_per_token_ms":11.727830545454545,"prompt_per_second":85.26726201612612,"predicted_n":283,"predicted_ms":3620.254794,"predicted_per_token_ms":12.792419766784452,"predicted_per_second":78.17129348714012,"sampler_ms":0.0,"parser_ms":2.80891,"draft_n":249,"draft_n_accepted":199}}
```


## Analysis

A few observations from the numbers:

1. Speculative decoding is the real winner here. Adding the DFlash draft model doubled llama.cpp's throughput from 42 → 84 tokens/sec. That's the ~2x improvement, and it comes from the draft model, not from Magnitude.

1. Magnitude did not beat llama.cpp in this test. With the same model + draft configuration, Magnitude clocked in at 78 tok/s vs llama.cpp's 84 tok/s—roughly 7% slower. Maybe it indeed is faster on meta and cuda. I don't have a machine to run them and ask one of my friends to check if it is faster meta.

1. The "2x faster" claim likely refers to a different comparison. It's possible Magnitude was comparing against llama.cpp without speculative decoding, in which case 78 vs 42 would indeed be close to 2x. But that's an apples-to-oranges comparison if llama.cpp is also capable of running the same draft model.