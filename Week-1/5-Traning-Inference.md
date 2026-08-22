# Training vs Inference

Two fundamental concepts in machine learning and LLMs are **training** and **inference**.

The simplest distinction is:

```text
Training
= Teaching the model

Inference
= Using the trained model
```

---

# Training

**Training** is the process of teaching a model to recognize patterns and perform tasks by exposing it to large amounts of data.

For an LLM, training involves processing enormous amounts of text and adjusting the model's parameters so that it becomes better at predicting tokens.

A simplified process looks like:

```text
Training Data
     ↓
Model
     ↓
Prediction
     ↓
Calculate Error
     ↓
Update Weights
     ↓
Repeat
```

For example, given:

```text
The capital of France is
```

the model attempts to predict:

```text
Paris
```

If the prediction is incorrect, the training process calculates how wrong the prediction was and adjusts the model's parameters.

This process is repeated on a massive scale.

---

## Parameters and Weights

During training, the model contains a large number of **parameters**.

These parameters are numerical values that determine how information flows through the neural network.

Training continuously adjusts these values.

```text
Before Training
     ↓
Random / Initial Parameters
     ↓
Training
     ↓
Updated Parameters
     ↓
Trained Model
```

The final parameters are commonly referred to as the model's **weights**.

The weights contain the learned patterns that allow the model to generate useful predictions.

---

# Gradient Descent

One of the fundamental optimization techniques used during neural network training is **gradient descent**.

The basic idea is:

```text
Model Prediction
      ↓
Calculate Loss
      ↓
Determine Direction of Improvement
      ↓
Adjust Parameters
      ↓
Repeat
```

The **loss function** measures how far the model's prediction is from the expected result.

The optimizer then adjusts the model's parameters to reduce the loss.

This happens repeatedly across huge amounts of training data.

---

# Backpropagation

**Backpropagation** is the process used to determine how the model's parameters contributed to the error.

A simplified training loop is:

```text
Input
  ↓
Forward Pass
  ↓
Prediction
  ↓
Loss Calculation
  ↓
Backpropagation
  ↓
Gradient Calculation
  ↓
Parameter Update
  ↓
Repeat
```

Modern LLM training involves extremely large-scale versions of these operations, distributed across many GPUs or other AI accelerators.

---

# Inference

**Inference** is the process of using a trained model to generate an output from new input.

Unlike training, the model's weights are normally **not updated during inference**.

For example:

```text
User:
What is Kubernetes?

        ↓

Trained LLM
        ↓

Generated Response
```

The model uses the knowledge encoded in its parameters to calculate the next tokens and generate the response.

---

# The Inference Process

For an LLM, inference generally works through **token prediction**.

Consider:

```text
The capital of France is
```

The model calculates probabilities for possible next tokens:

```text
Paris      → high probability
London     → lower probability
Berlin     → lower probability
Madrid     → lower probability
```

The model selects a token according to the configured generation strategy.

The process then continues:

```text
The capital of France is
              ↓
            Paris
              ↓
The capital of France is Paris
              ↓
        Next token...
```

This process continues until the model reaches an appropriate stopping condition.

---

# Training vs Inference

|                    | Training                                   | Inference                            |
| ------------------ | ------------------------------------------ | ------------------------------------ |
| Purpose            | Teach the model                            | Use the model                        |
| Data               | Training dataset                           | New user input                       |
| Weights updated?   | Yes                                        | Normally no                          |
| Computational cost | Extremely high                             | Much lower                           |
| Frequency          | Usually performed during model development | Happens every time the model is used |
| Example            | Creating a new LLM                         | Asking ChatGPT a question            |
| Main output        | Trained model weights                      | Generated response                   |

---

# Training Is Expensive

Training a frontier LLM from scratch requires enormous resources.

It can require:

* Large GPU clusters
* Massive datasets
* High-performance networking
* Large amounts of storage
* Significant electricity
* Specialized engineering infrastructure
* Long training periods

A simplified architecture might look like:

```text
Massive Dataset
       ↓
GPU Cluster
       ↓
Distributed Training
       ↓
Neural Network
       ↓
Updated Weights
       ↓
Trained LLM
```

This is why most application developers do not train frontier models from scratch.

Instead, they use existing models through APIs or download open-weight models.

---

# Inference Is What Happens When an Application Uses an LLM

Once a model has been trained, it can be deployed for inference.

For example:

```text
User
 ↓
Application
 ↓
API Request
 ↓
Trained LLM
 ↓
Inference
 ↓
Generated Response
 ↓
Application
 ↓
User
```

Every time an application sends a prompt to an LLM and receives a response, inference is taking place.

This means that when using ChatGPT, an API-based model, or a locally hosted model through Ollama, the model is performing **inference**.

---

# Training vs Inference Hardware

Training and inference can have different hardware requirements.

### Training

Training generally requires substantial computational resources because the model must:

* Process training data
* Calculate predictions
* Calculate loss
* Perform backpropagation
* Calculate gradients
* Update parameters

### Inference

Inference primarily needs to:

* Load the model
* Process the input
* Calculate the model's predictions
* Generate the output

Because inference does not normally update the model's parameters, it is considerably less computationally intensive than training the model from scratch.

However, inference can still be expensive at scale, especially for large models serving millions of users.

---

# Training, Fine-Tuning, and Inference

It is also important to distinguish **training**, **fine-tuning**, and **inference**.

```text
Pre-Training
     ↓
Base Model
     ↓
Fine-Tuning / Post-Training
     ↓
Specialized Model
     ↓
Inference
     ↓
Application
```

### Pre-Training

Creates the initial general-purpose model by training on massive datasets.

### Fine-Tuning

Further trains an existing model on a more specific dataset or objective.

### Inference

Uses the resulting model to generate predictions or responses.

---

# Simple Analogy

A useful analogy is learning for an exam.

```text
Training
= Studying

Model
= Student

Training Data
= Study Material

Weights
= What the student has learned

Inference
= Taking the exam
```

During studying, knowledge is being learned and updated.

During the exam, the student uses what was learned to answer questions.

Similarly:

```text
Training
→ Model learns

Inference
→ Model uses what it learned
```

---

# Key Takeaways

* **Training** is the process of learning patterns from data.
* Training modifies the model's parameters/weights.
* **Inference** is using a trained model to generate an output.
* Model weights are normally fixed during inference.
* Training a frontier model requires enormous computational resources.
* Inference happens every time a trained model processes a new request.
* Fine-tuning is additional training performed on an existing model.
* LLM applications primarily focus on **inference**, while model developers perform the large-scale training and post-training.

```text
Training
Data → Model → Loss → Parameter Updates
                     ↓
                Trained Model
                     ↓
Inference
Input → Trained Model → Output
```
