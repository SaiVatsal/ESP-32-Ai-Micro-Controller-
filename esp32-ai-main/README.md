# Running a 28.9M Parameter Language Model on an ESP32-S3
## IOT Project 
Running a language model on a microcontroller sounds almost impossible because these devices have very limited memory and processing power. However, with a carefully designed memory layout, it is possible.

This project demonstrates a 28.9-million-parameter language model running entirely on an ESP32-S3. It generates text directly on the device, without relying on cloud services or sending data to an external server. Generated text is displayed on a small screen connected to the board.

The model achieves approximately **9.88 tokens per second**. It fits on the microcontroller by storing most of its parameters in flash memory and loading only the required data during inference.

The same ESP32-S3 also runs a simulation based on part of a real fruit fly brain, containing more than 48,000 neurons and 9.46 million connections.

---

## 1. Language Model: Performance and Specifications

| Specification | Details |
|---|---|
| Total parameters | 28.9 million |
| Embedding parameters | Approximately 25 million |
| Microcontroller | ESP32-S3 |
| SRAM | 512 KB |
| PSRAM | 8 MB |
| Flash | 16 MB |
| Inference speed | 9.88 tokens/second |
| Compute time | 94.9 ms/token |
| Quantization | 4-bit |
| Model size | 14.9 MB |
| Internet connection | Not required |

## 2. How Does It Fit on a Microcontroller?

The main challenge is memory.

An ESP32-S3 has limited SRAM, which makes it difficult to load a conventional language model entirely into memory. Instead of loading everything at once, this project uses different types of memory according to how frequently the data is accessed.

The model's parameters are distributed across three memory levels:

```text
SRAM
  Fast memory with limited capacity.
  Stores activations and normalization weights.

PSRAM
  Larger working memory.
  Stores the main model components and output head.

FLASH
  Stores the large embedding lookup table.
  Only the required rows are accessed during generation.
```

Approximately 25 million parameters are stored in a flash-based lookup table. Rather than loading the entire table into RAM, the model retrieves only the rows needed for each token.

This approach is based on Google's Per-Layer Embeddings architecture, introduced with Gemma 3n.

It demonstrates how memory-efficient model designs can make on-device AI possible even on hardware with very limited resources.

## 3. What Can the Language Model Do?

The TinyStories version of the model is trained to generate short and simple stories.

It can produce basic sentences and maintain a reasonable level of coherence for short text generation.

However, it is important to understand its limitations. This model is not designed to behave like a general-purpose AI assistant. It cannot reliably answer questions, follow complex instructions, generate code, or provide factual information.

The main achievement is not the model's conversational ability. It is demonstrating that a language model with 28.9 million parameters can run directly on a small microcontroller.

---

# 4. Running a Fruit Fly Brain Simulation on the ESP32-S3

Alongside the language model, the same hardware can simulate part of a real fruit fly nervous system.

The simulation uses data from the MaleCNS v1.0 connectome, which maps connections within the male fruit fly's central nervous system.

The selected dataset contains:

- 48,311 neurons.
- 9,462,135 connections.
- 49,481,754 synaptic contacts.

The compressed graph occupies approximately 13.5 MiB of flash storage, with frequently accessed blocks cached in PSRAM.

Each simulation step processes the connections using both ESP32-S3 cores and its SIMD instructions.

The complete fruit fly nervous system is not included because its lossless representation requires more storage than the available flash memory.

## 5. Simulating an Escape Response

The simulation demonstrates how a fruit fly responds when a spider approaches.

A spider appears on the screen and gradually grows larger. This visual change activates the fly's looming detectors, including LC4 and LPLC2.

When the relevant escape neurons become sufficiently active, the simulated fly jumps away from the approaching spider.

One interesting aspect is that the escape behaviour does not require a separately trained AI model. It comes from the existing neural connections in the selected biological dataset.

The simulation uses a small configuration file to identify the relevant input neurons, output neurons, and activation threshold.

This makes the project an interesting example of how biological neural circuits can be explored using embedded hardware.

## 6. Fruit Fly Simulation Specifications

| Specification | Details |
|---|---|
| Neurons | 48,311 |
| Connections | 9,462,135 |
| Synaptic contacts | 49,481,754 |
| Compressed graph size | 13.5 MiB |
| Storage | Flash with PSRAM caching |
| Decision interval | Approximately 1.7 seconds |
| Training | No additional training required |

---

# 7. Running the Project

The project supports three downloadable model configurations:

| Model | Purpose |
|---|---|
| Barista | Espresso-related question answering |
| TinyStories | Short story generation |
| Fly | Fruit fly neural simulation |

### Downloading a Model

First, download the required model assets:

```bash
scripts/fetch_model.sh barista
```

Replace `barista` with `tinystories` or `fly` to download another configuration.

### Deploying to the ESP32-S3

After downloading, prepare and flash the firmware:

```bash
scripts/deploy.sh barista
```

The deployment process generates the required headers, runs validation checks, compiles the firmware, and flashes it onto the board.

Only one model configuration is installed on the board at a time. Deploying another configuration replaces the currently installed one.

### Model Integrity

The download process verifies the model files using their expected SHA-256 checksums and file sizes. It also checks the release metadata before installing the assets.

This helps prevent incomplete or modified downloads from being used accidentally.

The deployment process does not download model files. It works with the assets already available in the project directory.

---

# 8. Project Structure

The repository separates firmware, model assets, research code, and supporting documentation.

```text
project/
│
├── artifacts/
│   ├── barista/
│   ├── tinystories/
│   └── fly/
│
├── firmware/
│   ├── esp32_barista/
│   ├── esp32_tinystories/
│   └── esp32_fly/
│
├── src/
│
├── research/
│   ├── tinystories/
│   └── fly/
│
├── docs/
│   └── fly-connectome/
│
├── scripts/
│   ├── fetch_model.sh
│   └── deploy.sh
│
└── RESULTS.md
```

The `src/` directory contains the reusable implementation, while the `firmware/` directory contains the individual ESP32-S3 applications.

Research code for training, quantization, and performance evaluation is maintained separately.

---

# 9. Technologies and Research

The project brings together ideas from embedded systems, machine learning, memory optimization, and computational neuroscience.

Important references include:

- **TinyStories:** A dataset designed to train small language models to generate simple stories.
- **Per-Layer Embeddings:** A memory-efficient architecture introduced with Google's Gemma 3n.
- **llama2.c:** A lightweight C implementation that demonstrates how language models can be trained and executed with minimal dependencies.
- **MaleCNS v1.0:** A fruit fly neural connectome dataset used for the biological simulation.

The fruit fly connectome data is provided by the FlyEM Project and its research collaborators under the CC BY 4.0 license. See the attribution documentation for details about the selected data and modifications.

## 10. Conclusion

This project explores two different ways of using limited embedded hardware.

The first demonstrates that a 28.9-million-parameter language model can generate text directly on an ESP32-S3 without a cloud connection.

The second demonstrates that a large biological neural network can be represented and simulated using the same microcontroller.

Together, these experiments show how careful memory management, quantization, optimized computation, and efficient data storage can extend the capabilities of low-cost embedded devices.

**The goal is simple: explore how much useful computation can be performed directly on small hardware, without depending on powerful computers or cloud infrastructure.**

Done And Recreated by Sai Vatsal 
2500040224
