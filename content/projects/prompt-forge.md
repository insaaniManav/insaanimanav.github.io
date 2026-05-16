+++
title = "PromptForge"
date = "2024-08-01"
draft = false
cover = "images/promptforge.png"
tags = ["Go", "AI", "Open Source", "Prompt Engineering"]
+++

An AI prompt engineering workbench that generates, analyzes, and systematically tests prompts. Built with Go.

[github.com/insaaniManav/promptforge](https://github.com/insaaniManav/promptforge)

{{< image src="/images/promptforge.png" alt="PromptForge Interface" position="center" style="border-radius: 8px;" >}}

---

Most prompt tools are glorified text editors. PromptForge brings engineering discipline to the process — AI-assisted generation, deep analysis with scoring, and automatic test suite creation covering robustness, safety, and accuracy. Supports Claude, GPT-4, Azure OpenAI, and local Ollama models.

```bash
docker run -d -p 8080:8080 -e ANTHROPIC_API_KEY="your-key" ghcr.io/insaanimanav/prompt-forge:main
```
