# Gemini API skills

A library of skills for the Gemini API, SDK and model interactions.

## Installation

Install from this repository using `npx skills`.

```sh
# Show me what you got.
npx skills add google-gemini/gemini-skills --list

# Install a specific skill.
npx skills add google-gemini/gemini-skills --skill gemini-api-dev --global
```

Or use the Context7 skills CLI.

```sh
# Interactively browse and install skills.
npx ctx7 skills install /google-gemini/gemini-skills

# Install a specific skill.
npx ctx7 skills install /google-gemini/gemini-skills gemini-api-dev
```

## Skills in this repo

### gemini-api-dev

Skill for developing Gemini-powered apps. Provides the best practices for
building apps that use the Gemini API.

```sh
# Vercel skills
npx skills add google-gemini/gemini-skills --skill gemini-api-dev --global
```

```sh
# Context7 skills
npx ctx7 skills install /google-gemini/gemini-skills gemini-api-dev
```

### gemini-image-generation

使用Gemini API进行图像生成的专业技能。涵盖Nano Banana模型的使用、配置参数、最佳实践和完整的中文API参考，包括文本生成图像、图像编辑、多轮编辑等功能。

```sh
# Vercel skills
npx skills add google-gemini/gemini-skills --skill gemini-image-generation --global
```

```sh
# Context7 skills
npx ctx7 skills install /google-gemini/gemini-skills gemini-image-generation
```

## Disclaimer

This is not an officially supported Google product. This project is not
eligible for the [Google Open Source Software Vulnerability Rewards
Program](https://bughunters.google.com/open-source-security).
