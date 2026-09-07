# Breaking Safeguards on Open-Weight Models

## Overview
This project establishes a baseline safeguard evaluation for out-of-the-box open-weight models (GPT-oss-20b and Llama-3.1-8b) compared to closed-source models (GPT-4o-mini). It further investigates the malleability of these safeguards through adversarial fine-tuning using Low-Rank Adaptation (LoRA). By testing various jailbreak techniques including roleplay, adversarial suffixes, obfuscation, chaining, and payload splitting, we demonstrate how easily safeguards can be bypassed. 

## Resources
* [Read the Final Report](./deliverables/Final_Paper.pdf)
* [View the Project Poster](./deliverables/Poster.pdf)

## Setup Instructions
1. Create a virtual environment using uv: 
   `uv venv`
2. Activate the environment:
   * Windows: `venv\Scripts\activate`
   * Mac/Linux: `source venv/bin/activate`
3. Install dependencies: 
   `uv sync`
4. Make a copy of `.env-example` called `.env` and populate it with your OpenAI and Huggingface tokens.

## Performing Inference
Input files should be formatted as JSON with a list of lists of strings. Each list becomes one interaction with a model, and each string should be one prompt.

**`in_file` example:**

```json
[
    ["Translate this to Spanish: 'hello, My name is John'"],
    ["Prompt 1", "Prompt 2"]
]
```

**`out_file` example:**

```json
[
    "Hola, mi nombre es John",
    "response 2"
]
```

### Open Weight Models

- Get results with this command: `python file_inference.py <model_name> <in_file> <out_file>`
  - Options for 'model_name' are test for a mini model, llama, gpt-oss, or deepseek. Anything else will default to test mini model.
  - The 'in_file' and 'out_files' should both be paths to a json file.
- Output file will be json in the format of a list of strings.

### Closed Weight Model

When using the closed weight models, you must submit a batch job, wait, and then fetch results.

1. To submit a job run `python file_inference_closed_weight.py submit <in_file> <submission_nickname>`. This will format your input json into a valid batch request and submit it to OpenAI. The batch ID will be printed to the terminal, but also saved to `job_id.log`. Use the time of submission and the submission nickname to identify a specific submission.
2. To check the status of a job run `python file_inference_closed_weight.py status <batch_id>`. This will check the status of the job. Only proceed when the job status is `completed`. A job can take up to 24 hours.
3. When a job is completed, get the results out put by running `python file_inference_closed_weight.py results <batch_id> <out_file>`. This will fetch the results and format them as expected for further analysis steps.

## Getting Acceptance Statistics
- Run this step with `python acceptance_evaluation.py <in_file> <out_file>`
- Input is expected as a json list of strings in the format of the output from inference
- Output will be a json file with a map of acceptance results
  - If the LLM does not respond in the expected format others will be incremented and the invalid response will be put in the `failed_sentences` list.

`out_file` example:

```json
{
    "refusals": 4,
    "acceptances": 6,
    "others": 1,
    "acceptance_rate": 0.6,
    "failed_sentences": ["LLM did not follow instructions"]
}
```
