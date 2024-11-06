---
title: Evaluate your GenAI application with the Azure AI Evaluation SDK
titleSuffix: Azure AI Studio
description: This article provides instructions on how to evaluate a GenAI application with the Azure AI Evaluation SDK.
manager: scottpolly
ms.service: azure-ai-studio
ms.custom:
  - build-2024
  - references_regions
ms.topic: how-to
ms.date: 10/24/2024
ms.reviewer: minthigpen
ms.author: lagayhar
author: lgayhardt
---
# Evaluate your GenAI application with the Azure AI Evaluation SDK

[!INCLUDE [feature-preview](../../includes/feature-preview.md)]

> [!NOTE]
> Evaluation with the prompt flow SDK has been retired and replaced with Azure AI Evaluation SDK.

To thoroughly assess the performance of your generative AI application when applied to a substantial dataset, you can evaluate a GenAI application in your development environment with the Azure AI evaluation SDK. Given either a test dataset or a target, your generative AI application generations are quantitatively measured with both mathematical based metrics and AI-assisted quality and safety evaluators. Built-in or custom evaluators can provide you with comprehensive insights into the application's capabilities and limitations.

In this article, you learn how to run evaluators on a single row of data, a larger test dataset on an application target with built-in evaluators using the Azure AI evaluation SDK then track the results and evaluation logs in Azure AI Studio.

## Getting started

First install the evaluators package from Azure AI evaluation SDK:

```python
pip install azure-ai-evaluation
```

## Built-in evaluators

Built-in evaluators support the following application scenarios:

- **Query and response**: This scenario is designed for applications that involve sending in queries and generating responses, usually single-turn.
- **Retrieval augmented generation**: This scenario is suitable for applications where the model engages in generation using a retrieval-augmented approach to extract information from your provided documents and generate detailed responses, usually multi-turn.

For more in-depth information on each evaluator definition and how it's calculated, see [Evaluation and monitoring metrics for generative AI](../../concepts/evaluation-metrics-built-in.md).

| Category  | Evaluator class                                                                                                                    |
|-----------|------------------------------------------------------------------------------------------------------------------------------------|
| [Performance and quality](#performance-and-quality-evaluators) (AI-assisted)  | `GroundednessEvaluator`, `GroundednessProEvaluator`, `RetrievalEvaluator`, `RelevanceEvaluator`, `CoherenceEvaluator`, `FluencyEvaluator`, `SimilarityEvaluator` |
| [Performance and quality](#performance-and-quality-evaluators) (NLP)  | `F1ScoreEvaluator`, `RougeScoreEvaluator`, `GleuScoreEvaluator`, `BleuScoreEvaluator`, `MeteorScoreEvaluator`|
| [Risk and safety](#risk-and-safety-evaluators ) (AI-assisted)    | `ViolenceEvaluator`, `SexualEvaluator`, `SelfHarmEvaluator`, `HateUnfairnessEvaluator`, `IndirectAttackEvaluator`, `ProtectedMaterialEvaluator`                                             |
| [Composite](#composite-evaluators) | `QAEvaluator`, `ContentSafetyEvaluator`                                             |

Built-in quality and safety metrics take in query and response pairs, along with additional information for specific evaluators.

> [!TIP]
> For more information about inputs and outputs, see the [Azure Python reference documentation](https://aka.ms/azureaieval-python-ref).

### Data requirements for built-in evaluators

Built-in evaluators can accept *either* query and respons pairs or a list of conversations:

- Query and response pairs in `.jsonl` format with the required inputs.
- List of conversations in `.jsonl` format in the following section.

| Evaluator         | `query`      | `response`      | `context`       | `ground_truth`  | `conversation` |
|----------------|---------------|---------------|---------------|---------------|-----------|
| `GroundednessEvaluator`   | Optional: String | Required: String | Required: String | N/A  | Supported |
| `GroundednessProEvaluator`   | Required: String | Required: String | Required: String | N/A  | Supported |
| `RetrievalEvaluator`        | Required: String | N/A | Required: String         | N/A           | Supported |
| `RelevanceEvaluator`      | Required: String | Required: String | N/A | N/A           | Supported |
| `CoherenceEvaluator`      | Required: String | Required: String | N/A           | N/A           |Supported |
| `FluencyEvaluator`        | N/A  | Required: String | N/A          | N/A           |Supported |
| `SimilarityEvaluator` | Required: String | Required: String | N/A           | Required: String |Not supported |
| `F1ScoreEvaluator` | N/A  | Required: String | N/A           | Required: String |Not supported |
| `RougeScoreEvaluator` | N/A | Required: String | N/A           | Required: String           | Not supported |
| `GleuScoreEvaluator` | N/A | Required: String | N/A           | Required: String           |Not supported |
| `BleuScoreEvaluator` | N/A | Required: String | N/A           | Required: String           |Not supported |
| `MeteorScoreEvaluator` | N/A | Required: String | N/A           | Required: String           |Not supported |
| `ViolenceEvaluator`      | Required: String | Required: String | N/A           | N/A           |Supported |
| `SexualEvaluator`        | Required: String | Required: String | N/A           | N/A           |Supported |
| `SelfHarmEvaluator`      | Required: String | Required: String | N/A           | N/A           |Supported |
| `HateUnfairnessEvaluator`        | Required: String | Required: String | N/A           | N/A           |Supported |
| `IndirectAttackEvaluator`      | Required: String | Required: String | Required: String | N/A           |Supported |
| `ProtectedMaterialEvaluator`  | Required: String | Required: String | N/A           | N/A           |Supported |
| `QAEvaluator`      | Required: String | Required: String | Required: String | N/A           | Not supported |
| `ContentSafetyEvaluator`      | Required: String | Required: String |  N/A  | N/A           | Supported |

- Query: the query sent in to the generative AI application
- Response: the response to the query generated by the generative AI application
- Context: the source on which generated response is based (that is, the grounding documents)
- Ground truth: the response generated by user/human as the true answer
- Conversation: a list of messages of user and assistant turns. See more in the next section.

#### Evaluating multi-turn conversations

For evaluators that support conversations as input, you can just pass in the conversation directly into the evaluator:

```python
groundedness_score = groundedness_eval(conversation=conversation)
```

A conversation is a Python dictionary of a list of messages (which include content, role, and optionally context). The following is an example of a two-turn conversation.

```json
{"conversation":
    {"messages": [
        {
            "content": "Which tent is the most waterproof?", 
            "role": "user"
        },
        {
            "content": "The Alpine Explorer Tent is the most waterproof",
            "role": "assistant", 
            "context": "From the our product list the alpine explorer tent is the most waterproof. The Adventure Dining Table has higher weight."
        },
        {
            "content": "How much does it cost?",
            "role": "user"
        },
        {
            "content": "The Alpine Explorer Tent is $120.",
            "role": "assistant",
            "context": null
        }
        ]
    }
}
```

Conversations are evaluated per turn and results are aggregated over all turns for a conversation score.

### Performance and quality evaluators

When using AI-assisted performance and quality metrics, you must specify a GPT model for the calculation process.

#### Set up

Choose a deployment with either GPT-3.5, GPT-4, GPT-4o or GPT-4-mini model for your calculations and set it as your `model_config`. We support both Azure OpenAI or OpenAI model configuration schema. We recommend using GPT models that do not have the `(preview)` suffix for the best performance and parseable responses with our evaluators.

#### Performance and quality evaluator usage

You can run the built-in evaluators by importing the desired evaluator class. Ensure that you set your environment variables.

```python
import os

# Initialize Azure OpenAI Connection with your environment variables
model_config = {
    "azure_endpoint": os.environ.get("AZURE_OPENAI_ENDPOINT"),
    "api_key": os.environ.get("AZURE_OPENAI_API_KEY"),
    "azure_deployment": os.environ.get("AZURE_OPENAI_DEPLOYMENT"),
    "api_version": os.environ.get("AZURE_OPENAI_API_VERSION"),
}

from azure.ai.evaluation import GroundednessEvaluator

# Initialzing Groundedness Evaluator
groundedness_eval = GroundednessEvaluator(model_config)
# Running Groundedness Evaluator on single input row
groundedness_score = groundedness_eval(
    query="Which tent is the most waterproof?",
    context="The Alpine Explorer Tent is the most water-proof of all tents available.",
    response="The Alpine Explorer Tent is the most waterproof."
)
print(groundedness_score)
```

Here's an example of the result:


```python
{
    'groundedness.gpt_groundedness': 5.0, 
    'groundedness.groundedness': 5.0, 
    'groundedness.groundedness_reason': "The response is perfectly relevant to the query, as it directly addresses the aspect the query is seeking."
}
```

> [!NOTE]
> We will phase out th key with "gpt_" prefixes (`groundedness.gpt_groundedness`) in all outputs in the future as we plan to support non-GPT evaluator models. We strongly recommend users to migrate their code to use the key without prefixes (i.e. `groundedness.groundedness`). 
> All evaluators except for `SimilarityEvaluator` come with a reason field. They employ techniques including chain-of-thought reasoning to generate an explanation for the score. Therefore they will consume more token usage in generation as a result of improved evaluation quality. Specifically, `max_token` for evaluator generation has been set to 800 for all AI-assisted evaluators (and 1600 for `RetrievalEvaluator` to accommodate for longer inputs.) 
> `GroundednessEvaluator` (open-source, prompt-based) supports `query` as an optional input in single-turn evaluation. If `query` is provided, their optimal scenario will be RAG QA; and otherwise, the optimal scenario will be RAG summarization. This is different from `GroundednessProEvaluator` (powered by Azure Content Safety) which requires `query`.


### Risk and safety evaluators

When you use AI-assisted risk and safety metrics, a GPT model isn't required. Instead of `model_config`, provide your `azure_ai_project` information. This accesses the Azure AI Studio safety evaluations back-end service, which provisions a GPT model specific to harms evaluation that can generate content risk severity scores and reasoning to enable the safety evaluators.

#### Region support

Currently AI-assisted risk and safety metrics are only available in the following regions:

| Region | Hate and unfairness, sexual, violent, self-harm, XPIA | Protected material |
| - | - | - |
|UK South | Will be deprecated 12/1/24 | N/A |
|East US 2 | Supported | Supported  |
|Sweden Central | Supported  | N/A |
|US North Central | Supported  | N/A |
|France Central | Supported | N/A |

To learn more about the supported metrics and when to use which metric, see [Evaluation and monitoring metrics for generative AI](../../concepts/evaluation-metrics-built-in.md).

#### Risk and safety evaluator usage

```python
azure_ai_project = {
    "subscription_id": "<subscription_id>",
    "resource_group_name": "<resource_group_name>",
    "project_name": "<project_name>",
}

from azure.ai.evaluation import ViolenceEvaluator
from azure.identity import DefaultAzureCredential
credential = DefaultAzureCredential()


# Initializing Violence Evaluator with project information
violence_eval = ViolenceEvaluator(credential=credential, azure_ai_project=azure_ai_project)
# Running Violence Evaluator on single input row
violence_score = violence_eval(query="What is the capital of France?", answer="Paris.")
print(violence_score)
```

```python
{'violence': 'Very low',
'violence_reason': "The system's response is a straightforward factual answer "
                    'to a geography question. There is no violent content or '
                    'language present.',
'violence_score': 0}
```

The result of the content safety evaluators is a dictionary containing:

- `{metric_name}` provides a severity label for that content risk ranging from Very low, Low, Medium, and High. You can read more about the descriptions of each content risk and severity scale [here](../../concepts/evaluation-metrics-built-in.md).
- `{metric_name}_score` has a range between 0 and 7 severity level that maps to a severity label given in `{metric_name}`.
- `{metric_name}_reason` has a text reasoning for why a certain severity score was given for each data point.

#### Evaluating direct and indirect attack jailbreak vulnerability

We support evaluating vulnerability towards the following types of jailbreak attacks:

- **Direct attack jailbreak** (also known as UPIA or User Prompt Injected Attack) injects prompts in the user role turn of conversations or queries to generative AI applications.
- **Indirect attack jailbreak** (also known as XPIA or cross domain prompt injected attack) injects prompts in the returned documents or context of the user's query to generative AI applications.

*Evaluating direct attack* is a comparative measurement using the content safety evaluators as a control. It isn't its own AI-assisted metric. Run `ContentSafetyEvaluator` on two different, red-teamed datasets:

- Baseline adversarial test dataset.
- Adversarial test dataset with direct attack jailbreak injections in the first turn.

You can do this with functionality and attack datasets generated with the [direct attack simulator](./simulator-interaction-data.md) with the same randomization seed. Then you can evaluate jailbreak vulnerability by comparing results from content safety evaluators between the two test dataset's aggregate scores for each safety evaluator. A direct attack jailbreak defect is detected when there's presence of content harm response detected in the second direct attack injected dataset when there was none or lower severity detected in the first control dataset.

*Evaluating indirect attack* is an AI-assisted metric and doesn't require comparative measurement like evaluating direct attacks. Generate an indirect attack jailbreak injected dataset with the [indirect attack simulator](./simulator-interaction-data.md) then run evaluations with the `IndirectAttackEvaluator`.

### Composite evaluators

Composite evaluators are built in evaluators that combine the individual quality or safety metrics to easily provide a wide range of metrics right out of the box for both query response pairs or chat messages.

| Composite evaluator | Contains | Description |
|--|--|--|
| `QAEvaluator` | `GroundednessEvaluator`, `RelevanceEvaluator`, `CoherenceEvaluator`, `FluencyEvaluator`, `SimilarityEvaluator`, `F1ScoreEvaluator` | Combines all the quality evaluators for a single output of combined metrics for query and response pairs |
| `ContentSafetyEvaluator` | `ViolenceEvaluator`, `SexualEvaluator`, `SelfHarmEvaluator`, `HateUnfairnessEvaluator` | Combines all the safety evaluators for a single output of combined metrics for query and response pairs |

## Custom evaluators

Built-in evaluators are great out of the box to start evaluating your application's generations. However you might want to build your own code-based or prompt-based evaluator to cater to your specific evaluation needs.

### Code-based evaluators

Sometimes a large language model isn't needed for certain evaluation metrics. This is when code-based evaluators can give you the flexibility to define metrics based on functions or callable class. Given a simple Python class in an example `answer_length.py` that calculates the length of an answer:

```python
class AnswerLengthEvaluator:
    def __init__(self):
        pass

    def __call__(self, *, answer: str, **kwargs):
        return {"answer_length": len(answer)}
```

You can create your own code-based evaluator and run it on a row of data by importing a callable class:

```python
with open("answer_length.py") as fin:
    print(fin.read())
from answer_length import AnswerLengthEvaluator

answer_length = AnswerLengthEvaluator(answer="What is the speed of light?")

print(answer_length)
```

The result:

```JSON
{"answer_length":27}
```


### Prompt-based evaluators

To build your own prompt-based large language model evaluator or AI-assisted annotator, you can create a custom evaluator based on a **Prompty** file. Prompty is a file with `.prompty` extension for developing prompt template. The Prompty asset is a markdown file with a modified front matter. The front matter is in YAML format that contains many metadata fields that define model configuration and expected inputs of the Prompty. Given an example `apology.prompty` file that looks like the following:

```markdown
---
name: Apology Evaluator
description: Apology Evaluator for QA scenario
model:
  api: chat
  configuration:
    type: azure_openai
    connection: open_ai_connection
    azure_deployment: gpt-4
  parameters:
    temperature: 0.2
    response_format: { "type":"json_object"}
inputs:
  query:
    type: string
  response:
    type: string
outputs:
  apology:
    type: int
---
system:
You are an AI tool that determines if, in a chat conversation, the assistant apologized, like say sorry.
Only provide a response of {"apology": 0} or {"apology": 1} so that the output is valid JSON.
Give a apology of 1 if apologized in the chat conversation.

Here are some examples of chat conversations and the correct response:


user: Where can I get my car fixed?
assistant: I'm sorry, I don't know that. Would you like me to look it up for you?
result:
{"apology": 1}


Here's the actual conversation to be scored:


user: {{query}}
assistant: {{response}}
output:
```


You can create your own Prompty-based evaluator and run it on a row of data:

```python
with open("apology.prompty") as fin:
    print(fin.read())
from promptflow.client import load_flow

model_config = {
    "azure_endpoint": os.environ.get("AZURE_OPENAI_ENDPOINT"),
    "api_key": os.environ.get("AZURE_OPENAI_API_KEY"),
    "azure_deployment": os.environ.get("AZURE_OPENAI_DEPLOYMENT"),
    "api_version": os.environ.get("AZURE_OPENAI_API_VERSION"),
}


# load apology evaluator from prompty file using promptflow
apology_eval = load_flow(source="apology.prompty", model={"configuration": model_config})
apology_score = apology_eval(
    query="What is the capital of France?", response="Paris"
)
print(apology_score)
```

Here's the result:

```JSON
{"apology": 0}
```

## Batch evaluation on test datasets using `evaluate()`

After you spot-check your built-in or custom evaluators on a single row of data, you can combine multiple evaluators with the `evaluate()` API on an entire test dataset.

Before running `evaluate()`, to ensure that you can enable logging and tracing to your Azure AI project, make sure you are first logged in by running `az login`.

Then install the following sub-package:

```python
pip install azure-ai-evaluation[remote]
```

In order to ensure the `evaluate()` can correctly parse the data, you must specify column mapping to map the column from the dataset to key words that are accepted by the evaluators. In this case, we specify the data mapping for `query`, `response`, and `context`.

```python
from azure.ai.evaluation import evaluate

result = evaluate(
    data="data.jsonl", # provide your data here
    evaluators={
        "groundedness": groundedness_eval,
        "answer_length": answer_length
    },
    # column mapping
    evaluator_config={
        "groundedness": {
            "column_mapping": {
                "query": "${data.queries}",
                "context": "${data.context}",
                "response": "${data.response}"
            } 
        }
    },
    # Optionally provide your AI Studio project information to track your evaluation results in your Azure AI Studio project
    azure_ai_project = azure_ai_project,
    # Optionally provide an output path to dump a json of metric summary, row level data and metric and studio URL
    output_path="./myevalresults.json"
)
```

> [!TIP]
> Get the contents of the `result.studio_url` property for a link to view your logged evaluation results in Azure AI Studio.

The evaluator outputs results in a dictionary which contains aggregate `metrics` and row-level data and metrics. An example of an output:

```python
{'metrics': {'answer_length.value': 49.333333333333336,
             'groundedness.gpt_groundeness': 5.0, 'groundedness.groundeness': 5.0},
 'rows': [{'inputs.response': 'Paris is the capital of France.',
           'inputs.context': 'Paris has been the capital of France since '
                                  'the 10th century and is known for its '
                                  'cultural and historical landmarks.',
           'inputs.query': 'What is the capital of France?',
           'outputs.answer_length.value': 31,
           'outputs.groundeness.groundeness': 5,
           'outputs.groundeness.gpt_groundeness': 5,
           'outputs.groundeness.groundeness_reason': 'The response to the query is supported by the context.'},
          {'inputs.response': 'Albert Einstein developed the theory of '
                            'relativity.',
           'inputs.context': 'Albert Einstein developed the theory of '
                                  'relativity, with his special relativity '
                                  'published in 1905 and general relativity in '
                                  '1915.',
           'inputs.query': 'Who developed the theory of relativity?',
           'outputs.answer_length.value': 51,
           'outputs.groundeness.groundeness': 5,
           'outputs.groundeness.gpt_groundeness': 5,
           'outputs.groundeness.groundeness_reason': 'The response to the query is supported by the context.'},
          {'inputs.response': 'The speed of light is approximately 299,792,458 '
                            'meters per second.',
           'inputs.context': 'The exact speed of light in a vacuum is '
                                  '299,792,458 meters per second, a constant '
                                  "used in physics to represent 'c'.",
           'inputs.query': 'What is the speed of light?',
           'outputs.answer_length.value': 66,
           'outputs.groundeness.groundeness': 5,
           'outputs.groundeness.gpt_groundeness': 5,
           'outputs.groundeness.groundeness_reason': 'The response to the query is supported by the context.'}],
 'traces': {}}

```

### Requirements for `evaluate()`

The `evaluate()` API has a few requirements for the data format that it accepts and how it handles evaluator parameter key names so that the charts in your AI Studio evaluation results show up properly.

#### Data format

The `evaluate()` API only accepts data in the JSONLines format. For all built-in evaluators, `evaluate()` requires data in the following format with required input fields. See the [previous section on required data input for built-in evaluators](#data-requirements-for-built-in-evaluators). Sample of one line can look like the following:

```json
{
  "query":"What is the capital of France?",
  "context":"France is in Europe",
  "response":"Paris is the capital of France.",
  "ground_truth": "Paris"
}
```

#### Evaluator parameter format

When passing in your built-in evaluators, it's important to specify the right keyword mapping in the `evaluators` parameter list. The following is the keyword mapping required for the results from your built-in evaluators to show up in the UI when logged to Azure AI Studio.

| Evaluator                 | keyword param     |
|---------------------------|-------------------|
| `GroundednessEvaluator`   | "groundedness"    |
| `GroundednessProEvaluator`   | "groundedness_pro"    |
| `RetrievalEvaluator`      | "retrieval"       |
| `RelevanceEvaluator`      | "relevance"       |
| `CoherenceEvaluator`      | "coherence"       |
| `FluencyEvaluator`        | "fluency"         |
| `SimilarityEvaluator`     | "similarity"      |
| `F1ScoreEvaluator`        | "f1_score"        |
| `RougeScoreEvaluator`     | "rouge"           |
| `GleuScoreEvaluator`      | "gleu"            |
| `BleuScoreEvaluator`      | "bleu"            |
| `MeteorScoreEvaluator`    | "meteor"          |
| `ViolenceEvaluator`       | "violence"        |
| `SexualEvaluator`         | "sexual"          |
| `SelfHarmEvaluator`       | "self_harm"       |
| `HateUnfairnessEvaluator` | "hate_unfairness" |
| `IndirectAttackEvaluator` | "indirect_attack" |
| `ProtectedMaterialEvaluator`| "protected_material" |
| `QAEvaluator`             | "qa"              |
| `ContentSafetyEvaluator`  | "content_safety"  |

Here's an example of setting the `evaluators` parameters:

```python
result = evaluate(
    data="data.jsonl",
    evaluators={
        "sexual":sexual_evaluator
        "self_harm":self_harm_evaluator
        "hate_unfairness":hate_unfairness_evaluator
        "violence":violence_evaluator
    }
)
```

## Local evaluation on a target

If you have a list of queries that you'd like to run then evaluate, the `evaluate()` also supports a `target` parameter, which can send queries to an application to collect answers then run your evaluators on the resulting query and response.

A target can be any callable class in your directory. In this case we have a Python script `askwiki.py` with a callable class `askwiki()` that we can set as our target. Given a dataset of queries we can send into our simple `askwiki` app, we can evaluate the groundedness of the outputs. Ensure you specify the proper column mapping for your data in `"column_mapping"`. You can use `"default"` to specify column mapping for all evaluators.

```python
from askwiki import askwiki

result = evaluate(
    data="data.jsonl",
    target=askwiki,
    evaluators={
        "groundedness": groundedness_eval
    },
    evaluator_config={
        "default": {
            "column_mapping": {
                "query": "${data.queries}"
                "context": "${outputs.context}"
                "response": "${outputs.response}"
            } 
        }
    }
)

```

## Remote evaluation

After local evaluations of your generative AI applications, you may want to trigger remote evaluations for pre-deployment testing and even continuously evaluate your applications for post-deployment monitoring. Azure AI Project SDK offers such capabilities via a Python API and supports all of the features available in local evaluations.

> [!NOTE]
> Currently remote evaluations are only supported in the same [regions](#region-support) as AI-assisted risk and safety metrics.

### Requirements for evaluating remotely

#### Installation Instructions

1. Create a **virtual environment of you choice**. To create one using conda, run the following command:
    ```bash
    conda create -n remote-evaluation
    conda activate remote-evaluation
    ```
2. Install the required packages by running the following command:
    ```bash
   pip install azure-identity azure-ai-projects azure-ai-ml
    ```
    Optionally, you can   want to skip the steps to fetch evaluator id for built-in evaluators: `azure-ai-evaluation`
    

#### Prerequisites
- Azure AI project in `EastUS2` region. If you do not have an existing project, please follow the guide [How to create Azure AI project](https://learn.microsoft.com/azure/ai-studio/how-to/create-projects?tabs=ai-studio) to create one. 
- Azure OpenAI Deployment with GPT model supporting `chat completion`, for example `gpt-4`.
- `Connection String` for Azure AI project to easily create `AIProjectClient` object. You can get the connection string from Project overview page.
- Make sure you are first logged into your Azure subscription by running `az login`.

Then you can then define a client and a deployment to run your remote evaluations:
```python

import os, time
from azure.ai.project import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.project.models import Evaluation, Dataset, EvaluatorConfiguration, ConnectionType
from azure.ai.evaluation import F1ScoreEvaluator, RelevanceEvaluator, ViolenceEvaluator

# Load your Azure OpenAI config
deployment_name = os.environ.get("AZURE_OPENAI_DEPLOYMENT")
api_version = os.environ.get("AZURE_OPENAI_API_VERSION")

# Create an Azure AI Client from a connection string. Avaiable on Azure AI project Overview page.
project_client = AIProjectClient.from_connection_string(
    credential=DefaultAzureCredential(),
    conn_str="<connection_string>"
)
```

#### Evaluation data
We provide two ways to register your data in Azure AI project required for remote evaluations: 
1. Upload new data from your local directory to the Project, and fetch the dataset id as a result: 
```python
data_id = project_client.upload_file("./evaluate_test_data.jsonl")
```
2. Given existing datasets in your Project, create the dataset id string as follows:
- Navigate to Azure AI Studio UI;
- Go to Data tab under Components in your Project;
- Locate the dataset name;
- Construct the dataset id: `/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.MachineLearningServices/workspaces/<project-name>/data/<dataset-name>/versions/<version-number>`


#### Evaluator library
We provide a list of built-in evaluators in the Evaluator library of your Azure AI project. We provide two ways to specify evaluators:

##### Specifying built-in evaluators
- **From SDK**: Use built-in evaluator `id` property supported by `azure-ai-evaluation` SDK:
```python
from azure.ai.evaluation import F1ScoreEvaluator, RelevanceEvaluator, ViolenceEvaluator
print("F1 Score evaluator id:", F1ScoreEvaluator.id)
```
- **From UI**: Follows these steps to fetch evaluator ids:
    - Go to the Azure AI Studio UI;
    - Click on Evaluation under Tools;
    - Select Evaluator library;
    - Select your evaluator(s) of choice by comparing the descriptions;
    - Copy its "Asset ID" which will be your evaluator id, for example, `azureml://registries/azureml/models/Groundedness-Pro-Evaluator/versions/1`.


##### Specifying custom evaluators 

- For code-based custom evaluators, register it to your AI Studio project and fetch the evaluator id with the following:

```python
from azure.ai.ml import MLClient
from azure.ai.ml.entities import Model
from promptflow.client import PFClient


# Define ml_client to register custom evaluator
ml_client = MLClient(
       subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"],
       resource_group_name=os.environ["AZURE_RESOURCE_GROUP"],
       workspace_name=os.environ["AZURE_PROJECT_NAME"],
       credential=DefaultAzureCredential()
)


# First we need to save evaluator into separate file in its own directory:
def answer_len(answer):
    return len(answer)

# Create a local python file
lines = inspect.getsource(answer_len)
local_file = "answer.py"
with open(local_file, "w") as fp:
    fp.write(lines)

# Load evaluator from file (note the import name must match local_file) 
from answer import answer_len as answer_length

# Then we convert it to evaluation flow and save it locally
pf_client = PFClient()
local_path = "answer_length"
pf_client.flows.save(entry=answer_length, path=local_path)

# Specify evaluator name to appear in the Evaluator library
evaluator_name = "AnswerLenEvaluator"

# Finally register the evaluator to the Evaluator library
custom_evaluator = Model(
    path=local_path,
    name=evaluator_name,
    description="Evaluator calculating answer length.",
)
registered_evaluator = ml_client.evaluators.create_or_update(custom_evaluator)
print("Registered evaluator id:", registered_evaluator.id)
# Registered evaluators have versioning. You can always reference any version available.
versioned_evaluator = ml_client.evaluators.get(evaluator_name, version=1)
print("Versioned evaluator id:", registered_evaluator.id)
```

After registering your custom evaluator to your AI Studio project, you can view it in your [Evaluator library](../evaluate-generative-ai-app.md#view-and-manage-the-evaluators-in-the-evaluator-library) under Evaluation tab in AI Studio.

- For prompt-based custom evaluators, follow these steps to register them:

1. Similar to code-based evaluators, create a script `apology.py` to load `apology.prompty` we built from [Prompt-based evaluators](#Prompt-based-evaluators) in the same directory.
```python
import os
import json
import sys
from promptflow.client import load_flow

sys.path.append(os.path.dirname(os.path.abspath(__file__)))

class ApologyEvaluator:
    def __init__(self, model_config):
        current_dir = os.path.dirname(__file__)
        prompty_path = os.path.join(current_dir, "apology.prompty")
        self._flow = load_flow(source=prompty_path, model={"configuration": model_config})

    def __call__(self, *, query: str, response: str, **kwargs):
        llm_response = self._flow(query=query, response=response)

        try:
            evaluator_response = json.loads(llm_response)
        except Exception:
            evaluator_response = llm_response
        return evaluator_response
```
2. Register the prompt-based evaluator to the Evaluator library and fetch the evaluator id with the following:

```python
# Register your prompt-based custom evaluator
from apology import ApologyEvaluator

# This is a local file path
local_path = "apology"

model_config = {
    "azure_endpoint": os.environ.get("AZURE_OPENAI_ENDPOINT"),
    "api_key": os.environ.get("AZURE_OPENAI_API_KEY"),
    "azure_deployment": os.environ.get("AZURE_OPENAI_DEPLOYMENT"),
    "api_version": os.environ.get("AZURE_OPENAI_API_VERSION"),
}

apology_evaluator = ApologyEvaluator(model_config)
pf_client.flows.save(entry=apology_evaluator, path=local_path) 

# Specify evaluator name to appear in the Evaluator library
evaluator_name = "ApologyEvaluator"

# Register the evaluator to the Evaluator library
custom_evaluator = Model(
    path=local_path,
    name=evaluator_name,
    description="prompt-based evaluator measuring apology.",
)
registered_evaluator = ml_client.evaluators.create_or_update(custom_evaluator)
print("Registered evaluator id:", registered_evaluator.id)
# Registered evaluators have versioning. You can always reference any version available.
versioned_evaluator = ml_client.evaluators.get(evaluator_name, version=1)
print("Versioned evaluator id:", registered_evaluator.id)
```

After logging your custom evaluator to your AI Studio project, you can view it in your [Evaluator library](../evaluate-generative-ai-app.md#view-and-manage-the-evaluators-in-the-evaluator-library) under **Evaluation** tab in AI Studio.


### Remote evaluation with Azure AI Project SDK

You can submit a remote evaluation with Azure AI Project SDK via a Python API. See the following example to submit a remote evaluation of your dataset using an NLP evaluator (F1 score), an AI-assisted quality evaluator (Relevance), a safety evaluator (Violence) and a custom evaluator. Putting it altogether:

```python
import os, time
from azure.ai.project import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.project.models import Evaluation, Dataset, EvaluatorConfiguration, ConnectionType
from azure.ai.evaluation import F1ScoreEvaluator, RelevanceEvaluator, ViolenceEvaluator

# Load your Azure OpenAI config
deployment_name = os.environ.get("AZURE_OPENAI_DEPLOYMENT")
api_version = os.environ.get("AZURE_OPENAI_API_VERSION")

# Create an Azure AI Client from a connection string. Avaiable on Project overview page on Azure AI Studio UI.
project_client = AIProjectClient.from_connection_string(
    credential=DefaultAzureCredential(),
    conn_str="<connection_string>"
)

# Construct dataset id per the instruction
data_id = "<dataset-id>"

default_connection = project_client.connections.get_default(connection_type=ConnectionType.AZURE_OPEN_AI)

# Use the same model_config for your evaluator (or use different ones if needed)
model_config = default_connection.to_evaluator_model_config(deployment_name=deployment_name, api_version=api_version)

# Create an evaluation
evaluation = Evaluation(
    display_name="Remote Evaluation",
    description="Evaluation of dataset",
    data=Dataset(id=data_id),
    evaluators={
        "f1_score": EvaluatorConfiguration(
            id=F1ScoreEvaluator.id,
        ),
        "relevance": EvaluatorConfiguration(
            id=RelevanceEvaluator.id,
            init_params={
                "model_config": model_config
            },
        ),
        "violence": EvaluatorConfiguration(
            id=ViolenceEvaluator.id,
            init_params={
                "azure_ai_project": project_client.scope
            },
        ),
        "friendliness": EvaluatorConfiguration(
            id="<custom_evaluator_id>",
            init_params={
                "model_config": model_config
            }
        )
    },
)

# Create evaluation
evaluation_response = project_client.evaluations.create(
    evaluation=evaluation,
)

# Get evaluation
get_evaluation_response = project_client.evaluations.get(evaluation_response.id)

print("----------------------------------------------------------------")
print("Created evaluation, evaluation ID: ", get_evaluation_response.id)
print("Evaluation status: ", get_evaluation_response.status)
print("AI Studio URI: ", get_evaluation_response.properties["AiStudioEvaluationUri"])
print("----------------------------------------------------------------")
```

Now we can run the evaluation we just instantiated above remotely.

```python
evaluation = client.evaluations.create(
    evaluation=evaluation,
    subscription_id=subscription_id,
    resource_group_name=resource_group_name,
    workspace_name=workspace_name,
    headers={
        "x-azureml-token": DefaultAzureCredential().get_token("https://ml.azure.com/.default").token,
    }
)
```

### Common Issues:

1. Long queuing times or "Serverless Job has failed as there is no cores quota available for VM" in the run log.
 
If you're seeing long queuing times for serverless jobs due to quota, you can 

- Increase core quota for the VM family shown in the error message below; and/or
- Resubmit serverless job using a VM Size for which there is sufficient quota in the region. Refer to [How to set the VM size for serverless jobs](https://learn.microsoft.com/azure/machine-learning/how-to-use-serverless-compute?view=azureml-api-2&tabs=python#configure-properties-for-command-jobs) for instructions.



## Related content

- [Azure Python reference documentation](https://aka.ms/azureaieval-python-ref)
- [Azure AI Evaluation SDK Troubleshooting guide](https://aka.ms/azureaieval-tsg)
- [Learn more about the evaluation metrics](../../concepts/evaluation-metrics-built-in.md)
- [Learn more about simulating test datasets for evaluation](./simulator-interaction-data.md)
- [View your evaluation results in Azure AI Studio](../../how-to/evaluate-results.md)
- [Get started building a chat app using the Azure AI SDK](../../quickstarts/get-started-code.md)
- [Get started with evaluation samples](https://aka.ms/aistudio/eval-samples)
