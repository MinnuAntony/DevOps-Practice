

# JAMBA MODEL

Jamba is an LLM designed for natural language tasks: text generation, summarization, reasoning, code, and more.

The main motivation behind developing the **Jamba model** is to improve the efficiency of large language models without compromising output quality.

Most mainstream LLMs today are built on the **Transformer architecture**, which relies on the attention mechanism to maintain context. However:

* **Training time** grows *quadratically* with context length.
* **Inference time** grows *linearly* with context length per step.

In December 2023, the **Mamba architecture** was introduced to address these computational inefficiencies. Mamba compresses context into an efficient state—similar to a recurrent neural network—while also enabling parallel computation through a selection mechanism. This design provides much better computational efficiency, allowing long-context processing natively.

The drawback of pure Mamba models is that their robustness and output quality decline at large scales. To combine the strengths of both approaches, a **hybrid Transformer–Mamba architecture** was developed, resulting in the **Jamba model**. This architecture achieves high output quality with high computational efficiency, enabling greater throughput and a lower memory footprint.

Additionally, **Mixture of Experts (MoE)** is used in the Jamba model to further enhance throughput, efficiency, and quality.

The latest iteration is the **Jamba 1.5 model family**, which includes:

* **Jamba 1.5 Large**: 94 billion active parameters, 398 billion total parameters.
* **Jamba 1.5 Mini**: 12 billion active parameters, 52 billion total parameters.

Key features for developers include:

* Tool calling
* Structured JSON outputs
* Document input objects
* Streaming capabilities

The models are also multilingual, supporting **9 languages**, and are available as **open-weight models** on Hugging Face. They are widely accessible across platforms and frameworks, including **AWS, GCP, and others**.

---


## ARCHITECTURE

Its architecture combines three key ideas: Transformers, State Space Models (specifically Mamba), and Mixture of Experts. 
The goal is to balance reasoning power, efficiency for long sequences, and scalability."


Each Jamba block contains both Transformer-style attention layers and Mamba layers.
Transformer layers provide strong reasoning and in-context learning.
Mamba layers are much more efficient for long contexts because they don’t rely on quadratic attention.
Some feedforward layers are replaced by Mixture-of-Experts (MoE) layers, which expand capacity without activating the whole network.

- Layer Ratio

"In the released Jamba model, the ratio of Transformer to Mamba layers is 1:7. 
That means for every Transformer layer, there are seven Mamba layers. 
This ratio was found to give the best trade-off between efficiency and quality. So Jamba is mostly Mamba, but still keeps enough Transformers for reasoning."


- Mixture of Experts

For MoE:
Each MoE layer has 16 experts, but only the top 2 are activated per token.
This gives Jamba a total capacity of 52 billion parameters, but only 12 billion active parameters per token.
The result is a model that’s very large in potential, but cost-efficient in practice.

- Efficiency Features

One of Jamba’s main strengths is memory efficiency:
Transformers alone need very large KV caches (key-value memory) for long contexts.
At 256k tokens, LLaMA-2 70B requires 128GB just for the cache, Mixtral needs 32GB, but Jamba needs only 4GB.
This means Jamba can run on a single 80GB GPU, unlike many other models of its size.


## Version Timeline Summary

| Version                  | Release Date    | Highlights                                                                                                                                                                    |
| ------------------------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Jamba (base)**         | March 2024      | Hybrid MoE base model, not instruction-tuned ([arXiv][2], [Hugging Face][1], [AI21][3])                                                                                       |
| **Jamba 1.5 Mini/Large** | August 22, 2024 | Instruction-tuned; 256K context; fast; multilingual; enterprise-ready features ([AI21][4], [arXiv][8], [PR Newswire][5], [Google Cloud][6], [Amazon Web Services, Inc.][7])   |
| **Jamba 1.6**            | March 2025      | Enterprise-optimized improvements in speed and quality ([AI21][9], [Wikipedia][10])                                                                                           |
| **Jamba 1.7 Mini/Large** | \~July 2025     | Current API default; enhanced grounding & instruction following; same long context & hybrid architecture ([AI21 Labs Documentation][11], [Hugging Face][12], [Wikipedia][10]) |

---

---

###  Python Example — Jamba with Bedrock

```python
import boto3
import json

# Create a Bedrock client (use your AWS region where Bedrock is available)
bedrock_runtime = boto3.client(
    service_name="bedrock-runtime",
    region_name="us-east-1"  # Change to your region
)

# Choose the Jamba model (Mini or Large)
# Check AWS Bedrock console for available model IDs in your region
model_id = "ai21.jamba-1-5-large"   # Example model, can also use jamba-mini

# Your prompt / input text
prompt = "Summarize the benefits of using Kubernetes for deploying microservices."

# Request payload
body = {
    "inputText": prompt,
    "parameters": {
        "maxTokens": 200,
        "temperature": 0.7,
        "topP": 0.9
    }
}

# Invoke the model
response = bedrock_runtime.invoke_model(
    modelId=model_id,
    body=json.dumps(body)
)

# Parse response
result = json.loads(response["body"].read())
print("Generated text:")
print(result.get("outputText", "No output"))
```

---



