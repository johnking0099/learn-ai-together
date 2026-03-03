# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a documentation-only repository (no source code, no build/test/lint tooling). It serves as a knowledge-sharing resource for AI infrastructure topics, created for learning purposes.

## Content Structure

- `vllm-help/` — A comprehensive guide to `vllm serve` configuration parameters (based on vLLM v0.11.2)
  - `vllm-help/README.md` — Main document with all parameter tables organized into 12 sections (Model, Frontend, KV Cache, Scheduler, Parallelism, Load, LoRA, Multimodal, Structured Outputs, Observability, Compilation, Global)
  - `vllm-help/docs/` — Sub-documents with detailed parameter references, named by section number (e.g., `1-1-model-selection.md`, `5-1-core-parallel.md`)
  - `vllm-help/vllm-serve-help.output.txt` — Raw `vllm serve --help` output used as source material

## Writing Conventions

- 项目内容尽量使用中文（简体）描述，但专业术语保持英文（如 KV Cache、LoRA、tensor parallelism 等）
- The vLLM guide uses a three-tier exposure model for parameters: **普通用户** (all users), **高级用户** (advanced users), **平台管理** (platform-managed)
- Each parameter section includes product design suggestions (产品设计建议) aimed at platform engineers building model deployment UIs
- Parameter tables follow a consistent format: 参数 | 说明 | 默认值 | 暴露建议
