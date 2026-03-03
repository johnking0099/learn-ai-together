# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a documentation-only repository (no source code, no build/test/lint tooling). It serves as a knowledge-sharing resource for AI infrastructure topics, created for learning purposes.

## Content Structure

- `basic-concepts/` — AI 产品经理技术知识指南：面向从事大模型/图像生成模型部署服务化的产品经理，由浅入深的概念入门系列
  - `basic-concepts/README.md` — 索引页，介绍目标读者、学习路径、各章节链接
  - `basic-concepts/01-model-fundamentals.md` — 模型基础（参数量、精度、上下文长度、量化等）
  - `basic-concepts/02-deployment-and-serving.md` — 部署与服务化（GPU、推理框架、性能指标、KV Cache 等）
  - `basic-concepts/03-cost-and-resources.md` — 成本与资源规划（GPU 定价、模型选型、弹性伸缩、LoRA）
  - `basic-concepts/04-product-design-judgment.md` — 产品设计判断力（参数分层暴露、默认值设计）
  - `basic-concepts/05-image-generation.md` — 图像生成模型（扩散模型、ComfyUI、ControlNet 等）
  - `basic-concepts/06-boundaries.md` — 不需要深入的领域（划清 PM 的技术知识边界）
- `vllm-help/` — A comprehensive guide to `vllm serve` configuration parameters (based on vLLM v0.11.2)
  - `vllm-help/README.md` — Main document with all parameter tables organized into 12 sections (Model, Frontend, KV Cache, Scheduler, Parallelism, Load, LoRA, Multimodal, Structured Outputs, Observability, Compilation, Global)
  - `vllm-help/docs/` — Sub-documents with detailed parameter references, named by section number (e.g., `1-1-model-selection.md`, `5-1-core-parallel.md`)
  - `vllm-help/vllm-serve-help.output.txt` — Raw `vllm serve --help` output used as source material

## Writing Conventions

- 项目内容尽量使用中文（简体）描述，但专业术语保持英文（如 KV Cache、LoRA、tensor parallelism 等）
- The vLLM guide uses a three-tier exposure model for parameters: **普通用户** (all users), **高级用户** (advanced users), **平台管理** (platform-managed)
- Each parameter section includes product design suggestions (产品设计建议) aimed at platform engineers building model deployment UIs
- Parameter tables follow a consistent format: 参数 | 说明 | 默认值 | 暴露建议
