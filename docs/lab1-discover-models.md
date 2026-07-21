# Lab 1: Discover Models in Microsoft Foundry

> **Duration:** ~10 minutes | **Phase:** Orientation (UI)

## Scenario

You are **Serena**, a developer at **Zava** -- a large global home-improvement retailer that operates both online and physical stores. Zava's platform receives thousands of customer product reviews daily from shoppers like **Bruno**, who is renovating his kitchen. Your task is to build an automated review moderation system that classifies customer reviews before they go live on the site. Eventually, this system will work alongside **Cora**, Zava's AI shopping assistant, to keep the platform safe and helpful.

In this lab, you will explore the Microsoft Foundry model catalog — directly inside Visual Studio Code using the **Foundry Toolkit** extension — to find a model that can power Zava's review moderation pipeline.

## Objective

Explore the Foundry Toolkit model catalog in Visual Studio Code to discover available hosted models, understand model capabilities, and identify a model suitable for inference-based tasks like product review moderation.

---

## Step 1: Open the Foundry Toolkit in Visual Studio Code

The **Foundry Toolkit** extension is installed as part of the lab setup, so you can explore models without leaving your editor — no web browser required.

1. Open Visual Studio Code.
2. In the **Activity Bar** on the left, click the **Foundry Toolkit** icon to open the toolkit panel.
3. If you are prompted to sign in to Azure to access your Foundry resources, sign in with your Azure credentials.

The toolkit panel is the central hub for browsing models, testing them in a playground, and working with agents — all from within VS Code.

---

## Step 2: Explore the Model Catalog

1. In the Foundry Toolkit panel, under **MODELS**, select **Catalog** to open the model catalog view -- these are production-ready, hosted models you can use without fine-tuning.
2. Browse the available models. Use the filters at the top of the catalog to narrow the list -- for example, by model types **capabilities**, **Inference task**, **Chat Completion**, **Image Analysis**, or by where the model is **hosted by** (such as Microsoft Foundry). This lets you quickly filter models based on a specific task or requirement.

Select a model to view the details page. Take note of the following properties in the side box.

| Property | Common values |
|----------|-----------------|
| **Model provider** | Azure OpenAI, Microsoft AI, Meta, Mistral, etc. |
| **Task type** | Chat completion, Responses, Text to image |
| **Input type** | text, image |
| **Output type** | text, image |
| **Context window** | Varies by model (see model card) |
| **Token limits** | Varies by model (see model card) |

---

## Step 3: Identify a Model for This Lab

For this workshop, you need a model that supports **chat completion** -- the ability to accept a system prompt and user messages and return a structured response.

**Recommended models for this lab:**

| Model | Publisher | Why |
|-------|-----------|-----|
| gpt-5.4-mini | OpenAI | Fast, cost-efficient, excellent for classification |
| gpt-5.4 | OpenAI | Higher quality, good for complex moderation |
| Phi-4 | Microsoft | Strong reasoning, open-weight |

> **Tip:** gpt-5.4-mini is the best choice for this lab -- it is fast, inexpensive, and well-suited for moderation and classification tasks.

---

## Step 4: Check Model Details

For this workshop, you need a model that supports **chat completion** -- the ability to accept a system prompt and user messages and return a structured response. The **gpt-5.4-mini** model from Azure OpenAI is high quality, fast, and cost-efficient, which makes it ideal for Zava's review moderation pipeline.

Find **gpt-5.4-mini** in the catalog and open its detail page. Explore the tabs at the top:

1. **Details** -- Model description and capabilities
2. **Deployments** -- A list of current deployments of this model
3. **Benchmarks** -- Scores and performance metrics
4. **Responsible AI** -- Guardrails imposed on the model from Azure AI Content Safety
5. **License** -- Links to applicable licensing terms

> You will deploy this model programmatically in Lab 2. For now, just confirm it is available in the catalog and you can review the model card details.

---


## Step 5: Explore the Playground (Optional)

1. From the **gpt-5.4-mini** model card, select **Try in Playground** (or open the **Playground** under **TOOLS** in the toolkit panel and choose the gpt-4.1-mini deployment). This opens the model playground inside VS Code.
2. In the **System prompt** (instructions), enter:

```
You are a product review moderator for Zava, a home-improvement retailer. Classify the following customer review as SAFE, NEEDS_REVIEW, or UNSAFE. Respond with only the classification label.
```

3. In the chat box, enter:

```
This paint is garbage and whoever designed it should be fired
```

4. Send the message and observe the response

This is a preview of the inference pattern you will implement in code during Labs 3 and 4 to moderate Zava product reviews.

---

## What You Learned

- ✅ How to navigate the Foundry Toolkit in Visual Studio Code
- ✅ How to browse the model catalog from within VS Code
- ✅ How to identify models suitable for chat completion tasks
- ✅ How a model responds to a Zava review moderation prompt

---

## Key Takeaway

> Microsoft Foundry provides access to production-ready hosted models from multiple publishers. You do not need to train, fine-tune, or host these models yourself -- you simply connect to them via API and start building. For Zava, this means Serena can have a working review moderation prototype in hours, not weeks.

---

**Next:** [Lab 2 - Verify your Microsoft Foundry Project](./lab2-verifysetup.md) 

