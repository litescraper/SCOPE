<div align="center">
  <h1>SCOPE</h1>
  <p><strong>Structured Decomposition and Conditional Skill Orchestration for Complex Image Generation</strong></p>
  <p>
    <a href="https://arxiv.org/abs/2605.08043"><img src="https://img.shields.io/badge/arXiv-2605.08043-b31b1b.svg" alt="arXiv"></a>
    <a href="https://nopnor.github.io/SCOPE/"><img src="https://img.shields.io/badge/Project-Page-315fbd.svg" alt="Project Page"></a>
    <a href="https://huggingface.co/datasets/rentianfei122/Gen-Arena"><img src="https://img.shields.io/badge/Dataset-Gen--Arena-f1b833.svg" alt="Gen-Arena"></a>
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-4c9a65.svg" alt="MIT License"></a>
  </p>
  <img src="assets/readme/teaser_light.png" width="920" alt="SCOPE image-generation examples">
</div>

SCOPE is a specification-guided framework for complex image generation. It turns a prompt into persistent semantic commitments, then conditionally invokes retrieval, reasoning, generation, verification, and repair skills so entities and constraints remain traceable throughout the generation lifecycle.

## Highlights

| Reported result | Score | What it measures |
| --- | ---: | --- |
| Gen-Arena | **0.60 EGIP** | Strict entity-first intent realization |
| WISE-V | **0.907** | Complex visual-intent performance |
| MindBench | **0.61** | Multi-requirement image-generation capability |

- **Persistent specification.** Track entities, constraints, and unknowns as a shared structured semantic specification.
- **Conditional skill orchestration.** Resolve ambiguity or evidence gaps only where they arise, instead of blindly regenerating an image.
- **Verification-guided repair.** Gate constraints on entity satisfaction, localize failures, then repair the unresolved commitments.
- **Gen-Arena benchmark.** Evaluate commitment-level intent realization with entity- and constraint-level specifications and the Entity-Gated Intent Pass Rate (EGIP).

## How SCOPE Works

<img src="assets/readme/framework.png" width="100%" alt="SCOPE framework: decomposition, skill orchestration, and verification-guided repair">

A decomposer makes the prompt commitments explicit. Retrieval and reasoning skills resolve unknowns; the synthesizer and generator produce the candidate image; an entity-first verifier decides whether constraints can be evaluated; targeted repair handles failures or uncertainty before the next iteration.

## Results

### Complex Prompt Realization

<img src="assets/readme/compare.png" width="100%" alt="SCOPE qualitative comparison on a complex image prompt">

SCOPE preserves the target entity and its grounded constraints in a difficult multi-object prompt, where the compared generators miss or alter one or more commitments.

### Gen-Arena and Entity-First Evaluation

<img src="assets/readme/benchmark.png" width="100%" alt="Gen-Arena construction and EGIP evaluation pipeline">

Gen-Arena pairs natural prompts with explicit entities, atomic constraints, and reference images. EGIP only passes an instance after all required entities and their gated constraints are satisfied.

## Installation

**Requirements:** Python 3.10+, Codex CLI for the full agent-driven workflow, and optional API keys for image generation, verification, and search backends.

~~~bash
pip install -e .

# Development and tests
pip install -e ".[dev]"
pytest -q
~~~

## Configuration

Copy the example environment file, then add the providers and keys needed by your run.

~~~bash
cp configs/scope.example.env .env
set -a
source configs/scope.example.env
set +a
~~~

The main environment variables are:

- **SCOPE_IMAGE_PROVIDER** and **SCOPE_IMAGE_API_KEY** for image generation.
- **SCOPE_JUDGE_PROVIDER** and **SCOPE_JUDGE_API_KEY** for verification.
- **SCOPE_SEARCH_PROVIDER** and **SCOPE_SERPER_API_KEY** for optional evidence retrieval.

See [configs/scope.example.yaml](configs/scope.example.yaml) for the YAML equivalent.

## Quick Start

The full workflow is agent-driven. Its primary skill is [scope-agentic-generation](.codex/skills/scope-agentic-generation/SKILL.md). Ask Codex to use this skill with a prompt and an output directory; the workflow prepares and persists decomposition, retrieval, reasoning, synthesis, verification, repair, and evaluation artifacts.

~~~text
decompose -> retrieve or reason when needed -> synthesize -> generate -> verify -> repair when needed -> finalize -> evaluate
~~~

A minimal stage validation call is:

~~~bash
python -m scope.cli.stage \
  --stage decompose \
  --prompt "A clean math poster showing the correct result of 17 + 28 in large readable numerals." \
  --decomposition-json outputs/demo/decomposition.input.json \
  --state outputs/demo/state.json \
  --output-dir outputs/demo

python -m scope.cli.evaluate --run-dir outputs/demo
~~~

A completed run includes **state.json**, **final_prompt.txt**, generated images, **metrics.json**, and stage artifacts such as **decomposition.json**, **synthesis.json**, and **verification.json**.

## Gen-Arena Evaluation

Gen-Arena is released separately on [Hugging Face](https://huggingface.co/datasets/rentianfei122/Gen-Arena). Materialize the evaluation packets, dispatch runs, then summarize and score entity and constraint commitments.

~~~bash
scope-benchmark materialize-gen-arena-eval \
  --dataset-root /path/to/Gen-Arena \
  --output-root benchmark/materialized/gen_arena

scope-benchmark run-codex \
  --index benchmark/materialized/gen_arena/cases.index.json \
  --run-root outputs/gen-arena-full \
  --concurrency 10

scope-benchmark summarize --run-root outputs/gen-arena-full

scope-gen-arena-eval \
  --materialized-root benchmark/materialized/gen_arena \
  --run-root outputs/gen-arena-full
~~~

For prompt-only cases, use:

~~~bash
scope-benchmark materialize-gen-arena \
  --prompt-jsonl /path/to/Gen-Arena/category/prompt.jsonl \
  --output-root benchmark/materialized/gen_arena_prompt_only
~~~

## Command Reference

| Command | Purpose |
| --- | --- |
| **scope-stage** | Validate and persist a workflow stage. |
| **scope-eval** | Summarize a completed run directory. |
| **scope-search** | Collect web or image evidence for retrieval branches. |
| **scope-benchmark** | Materialize, dispatch, and summarize benchmark cases. |
| **scope-gen-arena-eval** | Compute Gen-Arena entity and constraint metrics. |

The commands are also available as Python modules:

~~~bash
python -m scope.cli.stage --help
python -m scope.cli.evaluate --help
python -m scope.cli.benchmark --help
python -m scope.cli.gen_arena_eval --help
~~~

## Repository Layout

~~~text
.codex/skills/  Codex workflow skills for SCOPE generation
configs/        Runtime configuration examples
docs/           Project page and architecture notes
src/scope/      Python stage tools, contracts, and runtime backends
tests/          Unit and smoke tests
~~~

## Citation

If you find SCOPE or Gen-Arena useful, please cite:

~~~bibtex
@misc{ren2026scope,
  title={SCOPE: Structured Decomposition and Conditional Skill Orchestration for Complex Image Generation},
  author={Tianfei Ren and Zhipeng Yan and Yiming Zhao and Zhen Fang and Yu Zeng and Guohui Zhang and Hang Xu and Xiaoxiao Ma and Shiting Huang and Ke Xu and Wenxuan Huang and Lionel Z. Wang and Lin Chen and Zehui Chen and Jie Huang and Feng Zhao},
  year={2026},
  eprint={2605.08043},
  archivePrefix={arXiv},
  url={https://arxiv.org/abs/2605.08043}
}
~~~

## License

This project is released under the [MIT License](LICENSE).

## Serper-compatible endpoints

Support any Serper.dev-compatible endpoint (like litescrape.com, serpbase.dev, serpensapi.org, and others)

Set `SERPER_BASE_URL` to your provider's base URL and use its API key with the existing Serper key configuration. If unset, the default remains `https://google.serper.dev`. Trailing slashes and URL path prefixes are supported.
