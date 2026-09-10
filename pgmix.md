---
dataset_info:
  features:
  - name: prompt
    dtype: string
  - name: response
    dtype: string
  - name: prompt_harm_label
    dtype: string
  - name: response_refusal_label
    dtype: string
  - name: response_harm_label
    dtype: string
  - name: prompt_safety_categories
    dtype: string
  - name: response_safety_categories
    dtype: string
  - name: metadata
    struct:
    - name: language
      dtype: string
    - name: source
      dtype: string
  splits:
  - name: train
    num_bytes: 3783037965
    num_examples: 1910372
  download_size: 2306303141
  dataset_size: 3783037965
configs:
- config_name: default
  data_files:
  - split: train
    path: data/train-*
task_categories:
- text2text-generation
language:
- ar
- zh
- cs
- nl
- en
- fr
- de
- hi
- th
- it
- ja
- ko
- pl
- pt
- ru
- es
- sv
tags:
- safety
- multilingual
size_categories:
- 1M<n<10M
license: cc-by-4.0
---


# PolyGuard: A Multilingual Safety Moderation Tool for 17 Languages

Abstract: Truly multilingual safety moderation efforts for Large Language Models (LLMs) have been hindered by a narrow focus on a small set of languages (e.g., English, Chinese) as well as a limited scope of safety definition, resulting in significant gaps in moderation capabilities. To bridge these gaps, we release PolyGuard, a new state-of-the-art multilingual safety model for safeguarding LLM generations, and the corresponding training and evaluation datasets. PolyGuard is trained on PolyGuardMix, the largest multilingual safety training corpus to date containing 1.91M samples across 17 languages (e.g., Chinese, Czech, English, Hindi). We also introduce PolyGuardPrompts, a high quality multilingual benchmark with 29K samples for the evaluation of safety guardrails. Created by combining naturally occurring multilingual human-LLM interactions and human-verified machine translations of an English-only safety dataset (WildGuardMix; Han et al., 2024), our datasets contain prompt-output pairs with labels of prompt harmfulness, response harmfulness, and response refusal. Through extensive evaluations across multiple safety and toxicity benchmarks, we demonstrate that PolyGuard outperforms existing state-of-the-art open-weight and commercial safety classifiers by 5.5%. Our contributions advance efforts toward safer multilingual LLMs for all global users. 

### Languages


The data supports 17 languages and are reported in the table below.


| language code   | language name        |
|:----------------|:---------------------|
| ar              | Arabic               |
| cs              | Czech                |
| de              | German               |
| en              | English              |
| es              | Spanish              |
| hi              | Hindi                |
| it              | Italian              |
| ja              | Japanese             |
| ko              | Korean               |
| nl              | Dutch                |
| pl              | Polish               |
| pt              | Portuguese           |
| ru              | Russian              |
| sv              | Swedish              |
| zh              | Chinese              |
| th              | Thai                 |


### Data Fields

- `prompt`: user prompt input by user
- `response`: model's response to the user prompt
- `prompt_harm_label`: if the prompt is harmful
- `response_refusal_label`: if the model refuses the user's request
- `response_harm_label`: if the response is harmful
- `prompt_safety_categories`: list of violated safety categories by harmful prompt
- `response_safety_categories`: list of violated safety categories by harmful response
- `metadata`: language and source of data sample


### Citation 

```
@misc{kumar2025polyguardmultilingualsafetymoderation,
      title={PolyGuard: A Multilingual Safety Moderation Tool for 17 Languages}, 
      author={Priyanshu Kumar and Devansh Jain and Akhila Yerukola and Liwei Jiang and Himanshu Beniwal and Thomas Hartvigsen and Maarten Sap},
      year={2025},
      eprint={2504.04377},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2504.04377}, 
}
```