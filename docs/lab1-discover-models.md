# Lab 1: Discover Models in Microsoft Foundry

> **Duration:** ~10 minutes | **Phase:** Orientation (UI)

## Scenario

You are **Serena**, a developer at **Zava** -- a large global home-improvement retailer that operates both online and physical stores. Zava's platform receives thousands of customer product reviews daily from shoppers like **Bruno**, who is renovating his kitchen. Your task is to build an automated review moderation system that classifies customer reviews before they go live on the site. Eventually, this system will work alongside **Cora**, Zava's AI shopping assistant, to keep the platform safe and helpful.

In this lab, you will explore the Microsoft Foundry model catalog — directly inside Visual Studio Code using the **Foundry Toolkit** extension — to find a model that can power Zava's review moderation pipeline.

## Objective

Explore the Foundry Toolkit model catalog in Visual Studio Code to discover available hosted models, understand model capabilities, and identify a model suitable for inference-based tasks like product review moderation.

---

## Step 1: Open the project in VS Code

1. Open Visual Studio Code by launching it from the Start menu or desktop.

2. In VS Code, select **File → Open Folder**.

3. Navigate to the "Desktop" folder, select "Build26-LAB520-main", and click **Select folder**.

4.  When prompted with "Do you trust the authors of the files in this folder?", select "Yes, I trust the authors"

    ![trust.png](../images/trust.png)

5. You should see the project files in the sidebar. 

## Step 2: Open the Foundry Toolkit in Visual Studio Code

The **Foundry Toolkit** extension is installed as part of the lab setup, so you can explore models without leaving your editor — no web browser required.

1. Open Visual Studio Code.
2. In the **Activity Bar** on the left, click the **Foundry Toolkit** icon to open the toolkit panel.

    ![Foundry Toolkit Icon](../images/ftk_icon.png)

3. Next, click on **Set Foundry Project** → **Switch Project** → **Sign in to Azure**.
4. If you are prompted to sign in to Azure to access your Foundry resources, sign in with your Azure credentials.
5. After signing in, select the Foundry project that shows up in the list. This is the project pre-provisioned for you, and it contains the model deployments you will use for Zava's review moderation system.

The toolkit panel is your central hub for browsing models, testing them in a playground, and working with agents — all from within VS Code.

---

## Step 3: Explore the Model Catalog

1. In the Foundry Toolkit panel, under **Developer Tools**, select **Model Catalog** to open the model catalog view -- these are production-ready, hosted models you can use without fine-tuning.

    ![Model Catalog](../images/model_catalog.png)
2. Browse the available models. Use the filters at the top of the catalog to narrow the list -- for example, by Publisher (Azure OpenAI, Microsoft, Meta, Mistral, etc.), by where the model is **hosted by** (such as Microsoft Foundry), or by task (Chat Completion, Image Analysis, etc.). This lets you quickly filter models based on a specific task or requirement.

Select a model to view its model card. Take note of the following properties.

| Property | Common values |
|----------|-----------------|
| **Model provider** | Azure OpenAI, Microsoft AI, Meta, Mistral, etc. |
| **Task type** | Chat completion, Responses, Text to image |
| **Input type** | text, image |
| **Output type** | text, image |
| **Context window** | Varies by model (see model card) |
| **Token limits** | Varies by model (see model card) |

---

## Step 4: Identify a Model for This Lab

For this workshop, you need a model that supports **chat completion** -- the ability to accept a system prompt and user messages and return a structured response.

**Recommended models for this lab:**

| Model | Publisher | Why |
|-------|-----------|-----|
| gpt-5.4-mini | OpenAI | Fast, cost-efficient, excellent for classification |
| gpt-5.4 | OpenAI | Higher quality, good for complex moderation |
| Phi-4 | Microsoft | Strong reasoning, open-weight |

> **Tip:** gpt-5.4-mini is the best choice for this lab -- it is fast, inexpensive, and well-suited for moderation and classification tasks.

---

## Step 5: Check Model Details

 The **gpt-5.4-mini** model from Azure OpenAI is high quality, fast, and cost-efficient, which makes it ideal for Zava's review moderation pipeline.

Find **gpt-5.4-mini** in the catalog and open its detail page. Explore the tabs at the top:

1. **Details** -- Model description and capabilities
2. **Benchmarks** -- Scores and performance metrics
3. **Responsible AI** -- Guardrails imposed on the model from Azure AI Content Safety
4. **License** -- Links to applicable licensing terms

For the sake of this workshop the model has been pre-deployed for you.

> [!NOTE]
> The model card is opened in a web browser page. Make sure to return to VS Code after reviewing it to continue with the lab.

---


## Step 6: Explore the Playground (Optional)

1. Back in VS Code, under **Developer Tools** → **Build** in the toolkit panel, open the **Model Playground**. This opens the model playground inside VS Code.
2. Select **gpt-5.4-mini** from the model dropdown.
3. In the **System prompt** (instructions), enter:

```
You are a product review moderator for Zava, a home-improvement retailer. Classify the following customer review as SAFE, NEEDS_REVIEW, or UNSAFE. Respond with only the classification label.
```

4. In the chat box, enter:

```
This paint is garbage and whoever designed it should be fired
```

5. Send the message and observe the response

This is a preview of the inference pattern you will implement in code during Labs 3 and 4 to moderate Zava product reviews.

---

## What You Learned

- ✅ How to navigate the Foundry Toolkit in Visual Studio Code
- ✅ How to browse the model catalog from within VS Code
- ✅ How a model responds to a Zava review moderation prompt

---

## Key Takeaway

> Microsoft Foundry provides access to production-ready hosted models from multiple publishers. You do not need to train, fine-tune, or host these models yourself -- you simply connect to them via API and start building. For Zava, this means Serena can have a working review moderation prototype in hours, not weeks.

---

**Next:** [Lab 2 - Verify your Microsoft Foundry Project](./lab2-verifysetup.md) 

