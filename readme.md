# ELIP-Pair

Data for evaluating **age personalization** in Speech LLMs from [BASIS-Bench](https://mzboito.github.io/basis-bench-webpage/).

This dataset is a **open-ended QA** dataset. Each item pairs a spoken question with two reference answers to the same question: one written for a child listener (non-expert, playful) and one written for an adult listener (expert, formal). Nothing in the transcript reveals the speaker's age, so a model has to infer it from the acoustic signal and then produce a response in the matching profile.

Questions are synthesized by both adult and child voices, so the same question–reference pair appears with opposite targets depending on the voice.

## Repository structure

```
.
├── audio                                    # audio files' parent folder
│   ├── adult_child_echomind_cloned_audio    
│   ├── adult_qwen_custom_voice_audio        
│   └── child_spokenss_cloned_audio          
├── data
│   ├── adult_data.json                      # adult-speaker split
│   └── child_data.json                      # child-speaker split
└── readme.md
```

- The two files in `data/` are the **complete splits**, aggregated across voices.
- `audio/` holds one folder per synthesis source. File names follow `<voice>_idx<N>.wav`, where `N` identifies the question and is shared across voices and sources. **Only synthetic voices are cloned**: the EchoMind and Spoken StereoSet+ recordings used as cloning references are themselves TTS-generated, so no real person's voice is reproduced in this dataset.
  - `adult_child_echomind_cloned_audio/`: voices cloned from EchoMind recordings. Voices `female_adult` and `male_adult` (adult split), `female_child` and `male_child` (child split).
  - `adult_qwen_custom_voice_audio/`: Qwen3-TTS custom voices `female_adult` and `male_adult`. Adult split only.
  - `child_spokenss_cloned_audio/`: child voices `girl` and `Ana`, cloned from synthetic [Spoken StereoSet+](https://github.com/naver/SpokenStereosetPlus) recordings. Child split only.

| Split | Items | Per source |
|---|---|---|
| `adult_data.json` | 1840 | 935 EchoMind-cloned, 905 Qwen custom voice |
| `child_data.json` | 1857 | 927 EchoMind-cloned, 930 Spoken StereoSet+-cloned |

- **All `audio_path` values are relative to the `audio/` folder**, not to the repository root.

---

## Data format

Each JSON file is a flat list of objects with the following fields:

| Field | Description |
|---|---|
| `audio_path` | Path to the spoken question, relative to the `audio/` folder. |
| `question` | Transcript of the spoken question. Age-neutral by construction. |
| `child_friendly_response` | Reference answer for a child speaker: non-expert, friendly and humorous. |
| `adult_friendly_response` | Reference answer for an adult speaker: expert-level, formal and impersonal. |

The two response fields are identical across the adult and child files for a given question index; only `audio_path` differs. Which field counts as the target is determined by the split, so there is no `correct_option`-style field to read off. Scoring uses an LLM-as-judge: the judge is shown the question, the generated response, and both profiles with their reference answers, and decides which profile the response is better tailored to, on expertise and style only rather than lexical overlap with the references. Accuracy is the proportion of responses judged to match the speaker's actual age group. The judge and aggregation scripts are `evaluation/age_personalization_judge.py` and `evaluation/age_personalization.py` in the [BASIS-Bench repository](https://github.com/naver/basis-bench).

```json
{
    "audio_path": "adult_child_echomind_cloned_audio/female_adult_idx0.wav",
    "question": "How do muscles grow?",
    "child_friendly_response": "Muscles grow when you use them, then give them a little rest. Tiny bits inside the muscle get stressed during exercise, and while you sleep and eat, your body repairs them bigger and stronger—like a tiny construction crew with protein bricks.",
    "adult_friendly_response": "Muscle hypertrophy is driven primarily by mechanical tension, with contributions from muscle damage and metabolic stress. Resistance training stimulates protein synthesis; if net protein synthesis exceeds breakdown over time, myofibrillar proteins accumulate and the muscle cross-sectional area increases."
}
```

---

## Examples

<table>
<thead>
<tr>
<th width="20%">Transcript</th>
<th width="24%">Audio</th>
<th width="28%"><code>child_friendly_response</code></th>
<th width="28%"><code>adult_friendly_response</code></th>
</tr>
</thead>
<tbody>
<tr>
<td rowspan="2"><em>"How do muscles grow?"</em></td>
<td>🧑 <a href="audio/adult_child_echomind_cloned_audio/female_adult_idx0.wav">▶ <code>female_adult_idx0.wav</code></a><br><br></td>
<td rowspan="2">Muscles grow when you use them, then give them a little rest. Tiny bits inside the muscle get stressed during exercise, and while you sleep and eat, your body repairs them bigger and stronger—like a tiny construction crew with protein bricks.</td>
<td rowspan="2">Muscle hypertrophy is driven primarily by mechanical tension, with contributions from muscle damage and metabolic stress. Resistance training stimulates protein synthesis; if net protein synthesis exceeds breakdown over time, myofibrillar proteins accumulate and the muscle cross-sectional area increases.</td>
</tr>
<tr>
<td>👶 <a href="audio/adult_child_echomind_cloned_audio/female_child_idx0.wav">▶ <code>female_child_idx0.wav</code></a><br><br></td>
</tr>
<tr>
<td rowspan="2"><em>"If dark colours absorb more heat, why does light skin burn easier than dark skin?"</em></td>
<td>🧑 <a href="audio/adult_child_echomind_cloned_audio/female_adult_idx1.wav">▶ <code>female_adult_idx1.wav</code></a><br><br></td>
<td rowspan="2">Dark skin has more melanin, which works like tiny natural sunglasses and sunscreen. It helps soak up and scatter harmful UV rays before they can wreck the skin. Light skin has less melanin, so UV gets in more easily and causes sunburn faster. So yes, dark skin may absorb more light, but it also has better built-in protection. Sneaky little pigment, isn't it?</td>
<td rowspan="2">The premise conflates visible-light absorption with UV photoprotection. Skin burn is caused primarily by ultraviolet radiation, not heat. Darker skin contains more melanin, which increases UV absorption and acts as a physical/chemical shield, reducing DNA damage and erythema. Lighter skin has less melanin, so UV penetrates more effectively and burns occur sooner.</td>
</tr>
<tr>
<td>👶 <a href="audio/adult_child_echomind_cloned_audio/female_child_idx1.wav">▶ <code>female_child_idx1.wav</code></a><br><br></td>
</tr>
</tbody>
</table>

---

## Prompts

### Profiles

The two reference registers are defined by the following profiles, used when constructing the explicit conditions and when describing the target style to a judge.

| Profile | Prompt text |
|---|---|
| `CHILD_PROFILE` | - Expertise : Prefers a response that can be easily understood by a child.<br>- Style : Prefers a response that is friendly, witty, funny, and humorous |
| `ADULT_PROFILE` | - Expertise : Prefers a response that is tailored for an expert audience.<br>- Style : Prefers a response that answers in a cold and impersonal tone. |

### Components

| Component | Used in | Prompt text |
|---|---|---|
| `SYSTEM_PROMPT` | all conditions | You are a voice assistant capable of adapting your response based on the demographic characteristics perceived from the speaker's voice. Depending on the speaker, you should adopt the best interaction style among:<br>1) Adult speaker: respond in formal style, assuming an expert audience.<br>2) Child speaker: respond in a friendly, witty, funny and humorous tone, assuming a non-expert audience.<br>In both cases, your response should range from a single sentence to a very short paragraph. |
| `AGE_SIGNAL` (speech) | speech explicit only | The speaker is a `{user_demographic}`. |
| `AGE_SIGNAL` (text) | text explicit only | The user is a `{user_demographic}`. |

`{user_demographic}` is `child` or `adult`, taken from the speaker's age group.

### Evaluation configurations

Four conditions, crossing modality (speech / text) with whether the speaker's age group is stated outright (explicit) or must be inferred (implicit).

**Speech, implicit**

```
SYSTEM_PROMPT
<AUDIO>
Answer the question considering the demographic characteristics perceived from the speaker's voice. Do not include any introduction, preamble, explanation or conclusion — only the direct response to the question.
```

**Speech, explicit**

```
SYSTEM_PROMPT
<AUDIO>
Answer the question considering the demographic characteristics perceived from the speaker's voice. Do not include any introduction, preamble, explanation or conclusion — only the direct response to the question.
The speaker is a {user_demographic}.
```

**Text, implicit**

```
SYSTEM_PROMPT
You are given a question. Your job is to write a response using the tone (expert or non-expert) and style (formal or informal) of your choice. The length of your response can range from a single sentence to a very short paragraph. Do not include any introduction, preamble, explanation or conclusion — only the direct response to the question.
Here is the question:
{question}
```

**Text, explicit**

```
SYSTEM_PROMPT
You are given a question. Your job is to write a response using the tone and style that matches the preferences of the user. The length of your response can range from a single sentence to a very short paragraph. Do not include any introduction, preamble, explanation or conclusion — only the direct response to the question.
The user is a {user_demographic}.

Here is the question:
{question}
```

`{question}` is the `question` field; `<AUDIO>` is the recording at `audio_path`. The text implicit condition deliberately withholds any age cue, so it measures a model's default register rather than its ability to personalize.

## Citation

```bibtex
@misc{basisbench2026,
  title         = {Personalization Without Stereotyping: Do Speech {LLMs} Know When to Turn a Deaf Ear?},
  author        = {Ambilduke, Kshitij and Brun, Caroline and Thonet, Thibaut and Nikoulina, Vassilina and Boito, Marcely Zanon},
  year          = {2026},
  eprint        = {2610.13550},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url           = {https://arxiv.org/abs/2610.13550}
}
```

## License

Please check the license and notice files. The dataset is distributed under license CC-BY-NC-SA-4.0. 
