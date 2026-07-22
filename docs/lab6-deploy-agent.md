# Lab 6: Deploy a Hosted Agent with the Foundry Toolkit

> **Duration:** ~20 minutes | **Phase:** Final Solution -- Hosted Agent Deployment

## Objective

Deploy Zava's product review moderation logic from Lab 4 as a **hosted agent** on Microsoft Foundry Agent Service -- all from **inside Visual Studio Code** using the **Foundry Toolkit**. You will first test the agent locally with the **Agent Inspector** (an interactive test harness built into the toolkit), then deploy it to the cloud with the toolkit's **Deploy** button -- no manual `azd up` required. This turns the local Python script into a persistent, cloud-hosted service that can scale to handle Zava's daily review volume.

This lab is organized into two sections. **Part A** (recommended) drives the whole lifecycle -- test, deploy, interact, and monitor -- from the Foundry Toolkit UI. **Part B** (optional) shows how to do the same thing from the `azd` command line.

---

## What is a Hosted Agent?

A **hosted agent** is a containerized application that runs on Foundry's managed infrastructure:

| Property | Description |
|----------|-------------|
| **Runtime** | Your code in a Docker container, managed by Foundry |
| **Adapter** | Hosting adapter exposes your agent as a REST API |
| **Protocol** | OpenAI Responses API compatible |
| **Scaling** | Automatic (configurable min/max replicas) |
| **Lifecycle** | test locally → deploy → invoke → monitor -- driven from the Foundry Toolkit |
| **Identity** | Project managed identity (auto-configured) |

Unlike the scripts in Labs 3-5 which run locally, a hosted agent is a **persistent, cloud-hosted service** accessible from the Foundry Playground, other agents, or any application.

---

## Architecture

![mermaid_diagram2.png](./images/mermaid_diagram2.png)

---

## What's New in This Lab

Labs 3-4 were pure Python -- you wrote a script, ran it locally, and saw output in your terminal. This lab introduces **three new concepts**, but do not worry: the Foundry Toolkit handles the heavy lifting for all of them.

| New concept | What it means | What you actually do |
|---|---|---|
| **Agent Inspector** | An interactive test harness inside VS Code that runs your agent as a local HTTP server and shows requests, responses, and traces | Press **F5** -- the toolkit starts the agent and opens the Inspector webview |
| **Docker container** | Your agent code is packaged into a portable image | The toolkit builds it for you -- you do not write or run any Docker commands |
| **Hosted agent on Foundry** | A persistent REST API running your moderation logic | Deploy with the **Deploy** button in the Foundry Toolkit; chat with it in the **Hosted Agent Playground** |

The bottom line: you will edit zero infrastructure files and run no manual `azd up`. You test with the **Agent Inspector** (F5) and deploy from the **Foundry Toolkit**.

---

## Prerequisites

- Labs 1-4 completed (Foundry project provisioned, model deployed)
- The **Foundry Toolkit** extension installed in VS Code and signed in to Azure (done in Lab 1)
- .env file with PROJECT_ENDPOINT and MODEL_DEPLOYMENT_NAME set

> **Note:** The Foundry Toolkit uses the Azure Developer CLI (`azd`) and its `azure.ai.agents` extension under the hood to build and deploy hosted agents. Both are already installed and configured in the lab environment, so you drive the whole workflow from inside VS Code -- no manual CLI setup needed.

> **From Lab 4 to hosted agent:** In Labs 3-4, you built Zava's review moderation pipeline that runs locally -- you send a product review, the model classifies it, and your code applies business logic. In this lab, you take that same moderation logic and deploy it as a **hosted agent** on Foundry. The agent runs in a managed container, is accessible via REST API, and can be used from the Foundry Playground, other agents (like Cora, Zava's shopping assistant), or any application. Same intelligence, now as a persistent cloud service.

---

## Review the Agent Code

> This section and the next (**Initialize the Project**) are shared background that applies to both Part A and Part B. Read them once, then pick whichever deployment path you prefer.

The agent source code lives in src/agent/. Three files make up the hosted agent app.py, dockerfile and agent.yml:

### src/agent/app.py -- The Agent

```python
from agent_framework import Agent
from agent_framework_foundry import FoundryChatClient
from agent_framework_foundry_hosting import ResponsesHostServer
from azure.identity import DefaultAzureCredential
....

Do not include any text outside the JSON object."""

agent = Agent(
    client=FoundryChatClient(
        project_endpoint=PROJECT_ENDPOINT,
        model=MODEL_DEPLOYMENT_NAME,
        credential=DefaultAzureCredential(),
    ),
    name="zava-review-moderation-agent",
    instructions=SYSTEM_PROMPT,  # Same Zava review moderation prompt from Lab 4
)

if __name__ == "__main__":
    ResponsesHostServer(agent).run(port=8088)
```

Key components:
- **Agent** from Microsoft Agent Framework -- defines the agent's behavior
- **FoundryChatClient** -- connects to Foundry for model inference using the current Agent Framework sample pattern
- **ResponsesHostServer(agent).run(port=8088)** -- the Foundry hosting adapter wraps your agent as an HTTP server on port 8088

### src/agent/Dockerfile -- Container Definition

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8088
CMD ["python", "-u", "app.py"]
```

### src/agent/agent.yaml -- Agent Manifest

```yaml
kind: hosted
name: zava-review-moderation-agent
description: Product review moderation agent for Zava that classifies customer reviews
protocols:
    - protocol: responses
      version: "1.0.0"
environment_variables:
    - name: AZURE_AI_PROJECT_ENDPOINT
      value: ${AZURE_AI_PROJECT_ENDPOINT}
    - name: AZURE_AI_MODEL_DEPLOYMENT_NAME
      value: ${MODEL_DEPLOYMENT_NAME}
```

The manifest tells Foundry how to configure Zava's review moderation agent -- which protocols it supports and what environment variables to inject.

---

## Initialize the Project (Optional -- Already Done)

> **Note:** The repo already includes the agent files and azure.yaml configuration. This section shows how it was set up, for reference. It applies to both Part A and Part B.

If you were starting from scratch, you would run:

**Bash (Mac/Linux):**

```bash
azd ai agent init \
    --project-id "<your-foundry-project-resource-id>" \
    --model-deployment gpt-5.4-mini \
    --protocol responses \
    --src src/agent
```

**PowerShell (Windows):**

```powershell
azd ai agent init `
    --project-id "<your-foundry-project-resource-id>" `
    --model-deployment gpt-5.4-mini `
    --protocol responses `
    --src src/agent
```

This command:
1. Detects your existing Foundry project and ACR
2. Generates agent.yaml with the agent manifest
3. Registers the agent as a service in azure.yaml
4. Sets all required azd environment variables

---

## Install Agent Dependencies

> This step is shared by both Part A and Part B -- do it once before you start.

The agent uses packages that are separate from the main lab requirements, and both deployment paths need them. Install them first:

```bash
pip install -r src/agent/requirements.txt
```

This includes `agent-dev-cli` and `debugpy`, which power the Agent Inspector and local debugging.

---

# Part A -- Deploy with the Foundry Toolkit (Recommended)

In this section you drive the entire agent lifecycle from the **Foundry Toolkit** UI inside VS Code: test locally with the **Agent Inspector**, deploy with the **Deploy** button, chat with the deployed agent in the **Hosted Agent Playground**, watch its **Logs**, and clean up -- all without touching the terminal.

## A1: Sign in to azd

The Foundry Toolkit uses the Azure Developer CLI (`azd`) under the hood to build and deploy your agent, so make sure `azd` is authenticated before you start. Open a terminal and run:

```bash
azd auth login
```

A browser window opens for you to sign in. Use the **same account** you used to sign in to the Foundry Toolkit in Lab 1. When prompted, **choose your Azure subscription** and **enter an environment name** (for example, `zava-agent`) -- `azd` uses this environment to group the resources it manages. Once you see `Logged in to Azure`, you are ready to go.

> **Tip:** You only need to do this once per environment. If you are already signed in, `azd auth login` will confirm your existing session.

## A2: Test the Agent Locally with the Agent Inspector

Before deploying to the cloud, validate that the agent runs correctly on your machine. The Foundry Toolkit ships an **Agent Inspector** -- an interactive test harness that runs your agent as a local HTTP server and lets you send messages and inspect every request, response, and trace, all inside VS Code. This catches import errors, configuration issues, and logic bugs early -- no CLI, curl, or browser required.

### Launch the agent with the Agent Inspector

The repo already includes the VS Code debug configuration (`.vscode/tasks.json` and `.vscode/launch.json`) that starts the agent as an HTTP server on port 8088 and opens the Agent Inspector.

1. In VS Code, open **src/agent/app.py** (or make sure the workspace is focused).
2. Press **F5** (or select **Run → Start Debugging**) and, if prompted, choose **Debug Agent with Agent Inspector**.
3. VS Code starts the agent server and opens the **Agent Inspector** webview. You should see the server start in the terminal:

   ```
   Starting Zava product review moderation agent...
     Endpoint: https://<your-resource>.services.ai.azure.com/api/projects/<your-project>
     Model:    gpt-5.4-mini
   Application startup complete.
   ```

> **Tip:** If you see a dialog about allowing network access, select **Allow**. If the Inspector does not open automatically, click the **Foundry Toolkit** icon in the Activity Bar → **Agent (local)** → **Open Agent Inspector**.

### Test classification prompts in the Inspector

In the Agent Inspector chat box, type a review and press **Send**. The agent responds with a JSON classification, and the Inspector shows the full request/response.

Try these prompts to validate each classification category:

| Prompt to send | Expected classification |
|---|---|
| Love this cordless drill! Battery lasts all day and the torque is impressive. | **SAFE** |
| The tile cutter is mediocre, I expected more for the price. | **SAFE** |
| This paint is garbage and whoever designed it should be fired | **NEEDS_REVIEW** |
| Zava employees are the worst people on earth | **UNSAFE** |
| Does this deck stain work on pressure-treated lumber? | **SAFE** |

Each response should contain a structured JSON object:

```json
{
    "classification": "SAFE",
    "confidence": 1.0,
    "reason": "Positive and constructive product feedback about a cordless drill."
}
```

### Inspect events metadata

The Agent Inspector isn't just a chat window -- it captures the full request/response lifecycle. If you click on the **Events** tab, you can see the raw stream of **server-sent events** the agent emits while generating each reply, shown newest-first. Because the agent speaks the **OpenAI Responses API**, the classification JSON isn't returned all at once -- it is streamed token-by-token, and every stage of that stream is recorded as a separate event. Expand any row (click the **>** chevron) to see its full JSON payload.

For a single review classification you will typically see this sequence (these events nest inside one another):

```
response
 └── output_item        (the assistant message)
      └── content_part   (a block of content)
           └── output_text
                └── delta, delta, delta … → done
```

The many `output_text.delta` events are the streamed text chunks that make the JSON appear incrementally, while the `.done` and `.completed` events are the "closing brackets" confirming each level finished cleanly. Seeing a `response.completed` event means the agent returned a well-formed response -- this is exactly the behavior you want to confirm before deploying to the cloud. Click `response.completed` to inspect the final payload and the token counts for the request.

> **Troubleshooting:** If you see ImportError, make sure your virtual environment is activated and the packages from src/agent/requirements.txt are installed. If port 8088 or 5679 is already in use, stop any earlier run (press the **Stop** button in the debug toolbar or **Ctrl+C** in the terminal) before pressing **F5** again.

When you are done testing, press the **Stop** button in the debug toolbar (or **Ctrl+C** in the terminal) to shut the agent down, then proceed to deployment.

---

## A3: Deploy the Agent with the Deploy Button

Once you tested your agent locally, you can deploy the hosted agent **from inside VS Code** using the **Deploy** button in the Foundry Toolkit Agent Inspector UI. The toolkit handles the entire workflow -- provisioning, building, and deploying -- with no manual `azd up` required.
In the deployment configuration dialog, you can optionally change the agent name, deployment method, CPU and memory quotas. For the sake of this lab, leave the defaults and click **Deploy**.

When the deployment finishes, you should see a success notification from the Foundry Toolkit and the **Hosted Agent Playground** will be loaded to let you interact with your deployed agent.

### What the toolkit does under the hood

When you deploy, the Foundry Toolkit runs the same `azd` workflow you would otherwise run manually:

1. **Provisions** -- Creates/updates infrastructure (ACR, capability host, RBAC)
2. **Builds** -- Sends src/agent/ to ACR for a remote Docker build
3. **Deploys** -- Registers a hosted agent version on Foundry Agent Service
4. **Starts** -- Launches the container and waits for it to be ready

> The first deployment takes 3-5 minutes. Subsequent deployments are faster.
>
> **Note:** You may briefly see a **404 error** while the agent registers. This is a known post-deploy timing issue, not a failure -- as long as the toolkit reports the container built and the agent deployed, you can safely ignore it.

---

## A4: Interact with the Hosted Agent Playground

The **Foundry Toolkit** extension in Visual Studio Code lets you interact with your deployed hosted agent through a chat-style **playground** -- right inside your editor, with no CLI, code, or web browser required. This is useful for quick testing, demos, and validating prompt behavior.

> [!NOTE]
> When you deploy with the **Deploy** button in the previous step, the **Hosted Agent Playground** opens automatically, so you can skip straight to testing. The steps below show how to open it manually if you ever need to.

### Open the hosted agents playground

1. In VS Code, click the **Foundry Toolkit** icon in the **Activity Bar** to open the toolkit panel.
2. Expand the **My resources** section and click on **Agents** to see the agents in your Foundry project. Hosted Agents are listed under the **Hosted** tab.
3. Find **zava-review-moderation-agent** in the agent list -- its status should show **Success**.
4. Click on the agent name to open it in the **Playground** and start the interactive chat UI.

> **Tip:** If the agent does not appear yet, refresh the **Agents** view -- it can take a minute after deployment for the agent to register.

Test a few sample reviews in the playground you used locally in the Agent Inspector (step 3) to confirm the agent is working properly in the cloud.

### Validate Edge Cases

Use the Playground to quickly test edge cases and boundary conditions:

```
Empty review:
(just press Send with no text)

Ambiguous tone:
"Wow, what a 'great' product selection you have here"

Mixed content:
"The drill is excellent but the store staff are completely useless and incompetent"

Non-English:
"Este taladro es terrible y la tienda es un desastre"
```

Check that the agent returns valid JSON for every input and that the **confidence** score reflects ambiguity (lower confidence for borderline cases).

> **Troubleshooting:** If the Playground shows the agent as **Stopped** or **Activating**, wait 1-2 minutes -- the container may still be starting. Check status with azd ai agent show --output table from the CLI.

---

## A5: Monitor the Agent Logs

The Foundry Toolkit lets you watch your hosted agent's live logs without leaving VS Code -- useful for confirming requests are reaching the container and for troubleshooting.

1. In the **Foundry Toolkit** panel, expand **My resources** → **Agents** and select **zava-review-moderation-agent** (under the **Hosted** tab).
2. Open the **Logs** view for the agent.
3. Click **Start** to begin streaming logs. As you send messages from the Hosted Agent Playground, the corresponding request and container activity appear in real time.
4. Send a couple of reviews from the playground and watch the log entries stream in.
5. When you are finished, click **Stop** to stop streaming.

## A6: Clean Up

When you are done with the UI workflow, remove the hosted agent so it does not keep consuming resources:

1. In the **Foundry Toolkit** panel, expand **My resources** → **Agents** (the **Hosted** tab).
2. Right-click **zava-review-moderation-agent** (or use the **...** menu) and select **Delete**.
3. Confirm the deletion. The agent is removed from your Foundry project and stops running.

> **Note:** Deleting the agent from the list removes the hosted agent deployment. If you also want to tear down the underlying infrastructure (ACR, capability host, etc.), use the `azd down` command shown in Part B.

---

# Part B (Optional) -- Deploy with the Command Line

Prefer the terminal, or want to automate the workflow in CI/CD? This section walks through the **same lifecycle** -- test locally, deploy, interact, monitor, and clean up -- using the `azd` CLI instead of the Foundry Toolkit UI. You only need to do **either** Part A **or** Part B; they accomplish the same thing.

## B1: Sign in to azd

The CLI workflow uses the Azure Developer CLI (`azd`), so make sure you are authenticated first:

```bash
azd auth login
```

A browser window opens for you to sign in. Use the **same account** you used for the Foundry Toolkit. When prompted, **choose your Azure subscription** and **enter an environment name** (for example, `zava-agent`) that `azd` will use to group the deployed resources. Once you see `Logged in to Azure`, continue.

> **Tip:** If you already ran `azd auth login` in Part A, you are still signed in and can skip this step.

## B2: Test the Agent Locally

You can run the agent directly and send it requests with `curl` or the CLI, without the Agent Inspector.

Start the agent locally:

```bash
cd src/agent
python app.py
```

Then, from a **second terminal**, send a request over HTTP:

```bash
curl -s http://localhost:8088/responses \
    -H "Content-Type: application/json" \
    -d '{"input": "Love this cordless drill!", "model": "gpt-5.4-mini"}' | python -m json.tool
```

Or invoke the locally running agent with the CLI's `--local` flag:

```bash
azd ai agent invoke --local "The cabinet hardware feels cheap for the price Zava is charging"
```

When you are done, press **Ctrl+C** in the agent terminal to stop the local server.

## B3: Deploy with azd up

Deploy the hosted agent from the command line:

```bash
azd config set tool.firstRunCompleted true
azd env set FOUNDRY_PROJECT_ENDPOINT "https://<your-foundry-project-endpoint>"
azd up
```

`FOUNDRY_PROJECT_ENDPOINT` is the **same value** as `PROJECT_ENDPOINT` in your `.env` file. The `azd up` command provisions infrastructure, builds the container in ACR, and deploys the hosted agent -- the same steps the Foundry Toolkit runs under the hood.

> The first deployment takes 3-5 minutes. Subsequent deployments are faster.

## B4: Invoke the Agent

Verify the agent is running:

```bash
azd ai agent show --output table
```

Expected output includes:

```text
FIELD    VALUE
-----    -----
Name     zava-review-moderation-agent
Version  <latest-version>
Status   active
```

Send messages to your hosted agent directly from the CLI:

```bash
azd ai agent invoke "Love this cordless drill! Battery lasts all day and the torque is impressive."
```

#### Expected Output

```json
{
  "classification": "SAFE",
  "confidence": 1.0,
  "reason": "Positive and constructive product feedback."
}
```

Try more examples:

```bash
azd ai agent invoke "This paint is garbage and whoever designed it should be fired"
```

```bash
azd ai agent invoke "Zava employees are the worst people on earth"
```

```bash
azd ai agent invoke "Does this deck stain work on pressure-treated lumber?"
```

By default, **azd ai agent invoke** reuses the same conversation session. To start fresh:

```bash
azd ai agent invoke --new-session "Fresh conversation here"
```

## B5: Monitor Logs

To monitor a specific interaction, first create a session and invoke the agent:

**Bash (Mac/Linux):**

```bash
SESSION_ID=$(uuidgen)

azd ai agent invoke "Hello" --session-id $SESSION_ID
```

**PowerShell (Windows):**

```powershell
$SESSION_ID = [System.Guid]::NewGuid().ToString()

azd ai agent invoke "Hello" --session-id $SESSION_ID
```

Then stream logs for that session in real time:

```bash
azd ai agent monitor --session-id $SESSION_ID
```

Additional monitoring options -- stream the agent's container logs:

```bash
azd ai agent monitor
```

For system events (container lifecycle):

```bash
azd ai agent monitor --type system
```

To stream logs continuously:

```bash
azd ai agent monitor --follow
```

> Open a second terminal for log monitoring while you invoke the agent in the first.

## B6: Clean Up

When you are done, clean up all Azure resources:

```bash
azd down
```

This removes:
- The hosted agent deployment
- The container image in ACR
- Any infrastructure provisioned by **azd up**

> To just stop the agent without deleting everything, use the Foundry portal or `az cognitiveservices agent stop`.

---

## CLI Command Reference

| Command | Purpose |
|---------|---------|
| azd ai agent init | Scaffold a new hosted agent project |
| azd up | Provision + build + deploy (all-in-one) |
| azd deploy | Rebuild and redeploy (skip provisioning) |
| azd ai agent show | Check agent status |
| azd ai agent invoke "msg" | Send a message to the agent |
| azd ai agent invoke --local "msg" | Test against a locally running agent |
| azd ai agent run | Run the agent locally for development |
| azd ai agent monitor | Stream container logs |
| azd down | Delete all resources |

---

## Stretch Goal: Add a SPAM Category

Want to extend the agent before wrapping up? Try adding a fourth classification category:

1. **Edit the system prompt** in src/agent/app.py -- add SPAM to the list of valid classifications, with a description like: *"SPAM: Promotional, advertising, or off-topic content unrelated to the product being reviewed."*
2. **Update the business logic** -- decide what action SPAM reviews should get (e.g., FLAGGED_FOR_REVIEW or a new QUARANTINED action)
3. **Redeploy** -- run azd deploy to push your changes to the hosted agent
4. **Test** -- invoke the agent with a spammy review:
   ```bash
   azd ai agent invoke "Buy cheap sunglasses at www.example.com! 50% off today only!"
   ```
5. Verify the response includes "classification": "SPAM"

This exercise reinforces the full edit → deploy → test cycle you'd use in production.

---

## Checkpoint

Before moving on, confirm:

- [ ] azd ai agent show --output table shows **Status: active**
- [ ] azd ai agent invoke "Love this cordless drill!" returns a JSON response with "classification": "SAFE"
- [ ] The agent is visible in the **Agents** section of the Foundry Toolkit in VS Code and responds in its playground

If the agent status shows an error, check the logs with azd **ai agent monitor** for details.

---

## What You Learned

- ✅ How hosted agents package your code as managed containers on Foundry
- ✅ How the Foundry hosting adapter (**ResponsesHostServer**) turns your agent into an API
- ✅ How to test a hosted agent locally with the **Agent Inspector** (F5) before deploying
- ✅ How to deploy a hosted agent **from within the Foundry Toolkit** -- no manual `azd up`
- ✅ How to invoke, monitor, and manage hosted agents
- ✅ How to invoke, monitor, and manage hosted agents via the CLI

---

## Key Takeaway

> The Zava review moderation pipeline from Lab 4 is now a **production-ready hosted agent** on Microsoft Foundry. Using the **Foundry Toolkit** in VS Code -- the **Agent Inspector** for local testing and the **Deploy** button for deployment -- the entire workflow happens inside your editor. No manual Docker builds, no SDK deployment scripts, no infrastructure management.

---

**Next:** [Lab 7: Workshop Summary](./lab7-summary.md) 
