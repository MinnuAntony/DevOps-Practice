# JAMBA MODEL

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

