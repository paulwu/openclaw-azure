# GitHub Copilot Chat Log

**Participant:** User
<br>**Assistant:** GitHub Copilot

<a name="table-of-contents"></a>
## Table of Contents

- [Request 1](#request-1): which prompt or agent can I use to help me explain how the code works in a pr...
- [Request 2](#request-2): @azure I've signed into my Microsoft account. Help me with my previous message.
- [Request 3](#request-3): Which of the prompts in this project folder can help me with explaining code ...

---

<a name="request-1"></a>
## Request 1 [^](#table-of-contents) < [>](#request-2)

### Participant

which prompt or agent can I use to help me explain how the code works in a project folder?

### Assistant

<details>
  <summary>Used 5 references</summary>
  <p>📄 file:README.md<br>☰ test.instructions.md<br>☰ copilot-instructions.md<br>⚙️ github.copilot.chat.codeGeneration.useInstructionFiles<br>☰ AGENTS.md</p>
</details>



You need to sign in to your Microsoft account to use GitHub Copilot for Azure (@azure).

After you've signed in, I can answer questions about Azure and your Azure resources and subscriptions. I can also assist with template selection, app deployment, diagnosing problems and more.

> $(wand) **New to Microsoft Azure?** \
> You may qualify for a $200 credit to use on Azure services when you sign up for a free account.
Already have a Microsoft account? [Sign in](command:@azure.signInAndRePrompt)

> *Response time: 0.24 seconds*
> <br>*Model: claude-opus-4.5*

---

<a name="request-2"></a>
## Request 2 [^](#table-of-contents) [<](#request-1) [>](#request-3)

### Participant

@azure I've signed into my Microsoft account. Help me with my previous message.

### Assistant

<details>
  <summary>Used 5 references</summary>
  <p>📄 file:README.md<br>☰ test.instructions.md<br>☰ copilot-instructions.md<br>⚙️ github.copilot.chat.codeGeneration.useInstructionFiles<br>☰ AGENTS.md</p>
</details>



To explain how the code works in a project folder using GitHub Copilot for Azure, you should use a custom agent or prompt designed for code explanation, such as those found in the [Awesome Prompts](https://github.com/github/awesome-copilot/tree/main/prompts) or [Awesome Agents](https://github.com/github/awesome-copilot/tree/main/agents) directories.

- Look for prompts ending in `.prompt.md` with a description related to code explanation or project walkthrough.
- Activate the agent or prompt in Copilot Chat or VS Code by using the `/` command (e.g., `/explain-code`).
- Ensure the prompt or agent follows the guidelines in [AGENTS.md](file:///home/devcontainers/awesome-copilot/AGENTS.md) and [copilot-instructions.md](file:///home/devcontainers/awesome-copilot/.github/copilot-instructions.md) for proper front matter and naming conventions.
- For more details, see [VS Code Copilot Customization Documentation](https://code.visualstudio.com/docs/copilot/copilot-customization) and [GitHub Copilot Chat Documentation](https://code.visualstudio.com/docs/copilot/chat/copilot-chat).
*Responses are currently scoped to the **Microsoft** tenant. [Select a different tenant](command:@azure.invokeChangeTenantSlashCommand).*

> *Response time: 24.71 seconds*
> <br>*Model: claude-opus-4.5*

---

<a name="request-3"></a>
## Request 3 [^](#table-of-contents) [<](#request-2) >

### Participant

Which of the prompts in this project folder can help me with explaining code in a project folder?

### Assistant

<details>
  <summary>Used 5 references</summary>
  <p>📄 file:README.md<br>☰ test.instructions.md<br>☰ copilot-instructions.md<br>⚙️ github.copilot.chat.codeGeneration.useInstructionFiles<br>☰ AGENTS.md</p>
</details>



Based on the workspace, the following prompts can help you explain code in a project folder:

| Prompt | Description |
| ------ | ----------- |
| [**Add Educational Comments**](prompts/add-educational-comments.prompt.md) | Adds educational comments to code files, explaining what code does and why, with configurable detail levels and knowledge levels for learning purposes. |
| [**Comment Code Generate A Tutorial**](prompts/comment-code-generate-a-tutorial.prompt.md) | Transforms Python scripts into beginner-friendly projects by refactoring code, adding instructional comments, and generating a complete markdown tutorial with setup instructions and example usage. |
| [**Repo Story Time**](prompts/repo-story-time.prompt.md) | Explores repository structure and explains the project's purpose, technologies, and key concepts by analyzing configuration files, documentation, and source directories. |
| [**Code Exemplars Blueprint Generator**](prompts/code-exemplars-blueprint-generator.prompt.md) | Analyzes a codebase to identify high-quality implementation examples and document best practices, patterns, and conventions used throughout the project. |
| [**Technology Stack Blueprint Generator**](prompts/technology-stack-blueprint-generator.prompt.md) | Generates comprehensive documentation of the technology stack, implementation patterns, and usage examples to help understand how a project is built. |
| [**Architecture Blueprint Generator**](prompts/architecture-blueprint-generator.prompt.md) | Creates detailed documentation of project architecture, component relationships, and design patterns to explain how the codebase is structured. |

The **Add Educational Comments** and **Comment Code Generate A Tutorial** prompts are most directly focused on explaining code, while the blueprint generators provide higher-level project understanding.

> *Response time: 12.24 seconds*
> <br>*Model: claude-opus-4.5*
