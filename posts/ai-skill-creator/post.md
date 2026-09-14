---
title: OpenAI官方的一份如何写Skill、优化Skill的教程
tags: [ai, skill]
date: 2025-12-06
private: false
template: post
---

# 用 Evals 系统化测试 Agent Skills

一份实用指南：如何让 Agent Skills 变成可以持续测试、评分和改进的东西。

作者：Dominik Kundel、Gabriel Chua

当你不断迭代一个供 Codex 这类 Agent 使用的 Skill 时，很难判断自己究竟是在真正改进它，还是仅仅改变了它的行为。一个版本似乎更快，另一个看起来更可靠，但随后某个回归问题又悄悄出现：Skill 没有被触发、跳过了某个必要步骤，或者留下了多余文件。

从本质上说，Skill 是为 LLM 组织起来的一组 Prompt 和指令。要想长期、可靠地改进一个 Skill，最有效的方法，就是像评估其他 LLM 应用中的 Prompt 一样去评估它。

*Evals* 是 *evaluations*（评估）的简称，用来检查模型的输出，以及它为了得到这些输出所执行的步骤，是否符合你的预期。与其问“这个版本是不是感觉更好了？”——或者纯凭感觉判断——不如通过 Eval 提出一些具体问题，例如：

* Agent 是否调用了这个 Skill？
* 它是否执行了预期的命令？
* 它生成的输出是否遵循你关心的约定？

具体来说，一个 Eval 可以概括为：

Prompt → 捕获的一次运行结果（trace + artifacts）→ 一小组检查 → 一个可以随时间比较的分数。

在实践中，针对 Agent Skills 的 Eval 很像轻量级的端到端测试：运行 Agent，记录发生了什么，然后按照一小组规则给结果打分。

本文将介绍一种清晰的 Codex 测试模式：先定义“成功”是什么，再加入确定性检查和基于评分标准（rubric）的评估，让改进和回归都变得清晰可见。

## 1. 在编写 Skill 之前，先定义什么叫成功

在真正开始写 Skill 之前，先明确写下什么叫“成功”，而且这些定义必须是可以实际衡量的。

一种实用的思考方式，是把检查项分成几类：

* **结果目标（Outcome goals）：** 任务是否完成？应用能否运行？
* **过程目标（Process goals）：** Codex 是否调用了 Skill，并按照你预期的工具和步骤执行？
* **风格目标（Style goals）：** 输出是否遵循你要求的规范？
* **效率目标（Efficiency goals）：** 是否在没有无效折腾的情况下完成任务，例如避免不必要的命令或过量的 Token 消耗？

这个列表应该保持精简，只关注必须通过的检查项。目标不是一开始就把所有偏好都编码进去，而是捕获那些你最在意的行为。

例如，本文会评估一个用于搭建 Demo 应用的 Skill。其中有些检查非常具体：它有没有执行 `npm install`？有没有创建 `package.json`？

与此同时，我们还会结合一个结构化的风格评分标准，用来评估代码约定和页面布局等内容。

这种组合是有意为之。你需要的是快速、针对性强的信号，让具体的回归问题尽早暴露出来，而不是等到最后只得到一个笼统的“通过/失败”结果。

## 2. 创建 Skill

一个 Codex Skill 本质上是一个目录，其中包含一个 `SKILL.md` 文件。

该文件由两部分组成：

* YAML front matter，其中包含 `name` 和 `description`
* 定义 Skill 行为的 Markdown 指令，以及可选的资源和脚本

`name` 和 `description` 的重要性可能比你想象中更高。

它们是 Codex 判断以下问题时最主要的信号：

* 是否应该调用这个 Skill
* 什么时候应该把 `SKILL.md` 的其余内容注入 Agent 的上下文

如果这些字段过于模糊，或者承担了太多含义，Skill 就无法稳定地被正确触发。

最快的入门方法，是使用 Codex 内置的 Skill Creator——它本身其实也是一个 Skill：

```text
$skill-creator
```

Creator 会询问你：

* 这个 Skill 是做什么的
* 它应该在什么情况下触发
* 它是纯指令型，还是由脚本支持

默认建议采用纯指令型。

### 一个示例 Skill

本文使用一个刻意保持最小化的例子：创建一个 Skill，以可预测、可重复的方式搭建一个小型 React Demo 应用。

这个 Skill 将执行以下工作：

* 使用 Vite 的 React + TypeScript 模板搭建项目
* 使用官方 Vite 插件方式配置 Tailwind CSS
* 强制采用最小且一致的文件结构
* 明确定义“完成标准”（definition of done），从而使成功与否容易评估

下面是一份简洁的草稿，可以放到：

* `.codex/skills/setup-demo-app/SKILL.md`，作用域为当前仓库
* `~/.codex/skills/setup-demo-app/SKILL.md`，作用域为当前用户

```markdown
---
name: setup-demo-app
description: Scaffold a Vite + React + Tailwind demo app with a small, consistent project structure.
---

## When to use this

Use when you need a fresh demo app for quick UI experiments or reproductions.

## What to build

Create a Vite React TypeScript app and configure Tailwind. Keep it minimal.

Project structure after setup:

- src/
  - main.tsx (entry)
  - App.tsx (root UI)
  - components/
    - Header.tsx
    - Card.tsx
  - index.css (Tailwind import)
- index.html
- package.json

Style requirements:

- TypeScript components
- Functional components only
- Tailwind classes for styling (no CSS modules)
- No extra UI libraries

## Steps

1. Scaffold with Vite using the React TS template:
   npm create vite@latest demo-app -- --template react-ts

2. Install dependencies:
   cd demo-app
   npm install

3. Install and configure Tailwind using the Vite plugin.
   - npm install tailwindcss @tailwindcss/vite
   - Add the tailwind plugin to vite.config.ts
   - In src/index.css, replace contents with:
     @import "tailwindcss";

4. Implement the minimal UI:
   - Header: app title and short subtitle
   - Card: reusable card container
   - App: render Header + 2 Cards with placeholder text

## Definition of done

- npm run dev starts successfully
- package.json exists
- src/components/Header.tsx and src/components/Card.tsx exist
```

这个示例 Skill 是有意采用明确立场、加入具体约束的。

原因很简单：如果没有明确约束，就没有任何具体内容可供评估。

## 3. 手动触发 Skill，以暴露隐藏假设

由于 Skill 是否被调用高度依赖 `SKILL.md` 中的 `name` 和 `description`，首先要验证的事情，就是 `setup-demo-app` 是否会在你预期的场景中被触发。

在开发早期，可以在真实仓库或临时目录中显式激活 Skill：

* 使用 `/skills` 斜杠命令
* 使用 `$` 前缀直接引用 Skill

然后观察它在哪里出问题。

这个过程可以帮助你发现各种遗漏，例如：

* Skill 完全没有触发
* Skill 触发得过于积极
* Skill 虽然运行了，但偏离了你设定的步骤

在这个阶段，你并不是在优化速度或者打磨细节。

你要寻找的是 Skill 中隐藏的假设，例如：

* **触发假设（Triggering assumptions）：**
  类似“搭建一个快速的 React Demo”这样的 Prompt，本来应该触发 `setup-demo-app`，但实际上没有触发。或者更通用的请求，比如“添加 Tailwind 样式”，却意外触发了这个 Skill。

* **环境假设（Environment assumptions）：**
  Skill 假设自己运行在一个空目录中，或者默认认为系统中已经有 `npm`，并且优先使用 `npm` 而不是其他包管理器。

* **执行假设（Execution assumptions）：**
  Agent 跳过了 `npm install`，因为它假设依赖已经安装；或者在 Vite 项目还不存在之前，就开始配置 Tailwind。

当你准备把这些运行过程变得可重复时，就可以切换到 `codex exec`。

它专门针对自动化和 CI 场景设计：

* 进度信息会流式写入 `stderr`
* 只有最终结果写入 `stdout`

因此更方便通过脚本执行、捕获和检查运行结果。

默认情况下，`codex exec` 会运行在受限的沙箱环境中。

如果任务需要写入文件，可以使用 `--full-auto`。

总体原则是，尤其在自动化环境中，应始终使用完成任务所需的最小权限。

一个基本的手动运行示例如下：

```bash
codex exec --full-auto \
  'Use the $setup-demo-app skill to create the project in this directory.'
```

第一次实际运行的主要目的并不是验证最终正确性，而是发现边界情况。

你在这个阶段做出的每一个手动修复，例如：

* 补上缺失的 `npm install`
* 修正 Tailwind 配置
* 收紧 Skill 的触发描述

都可以变成未来的一条 Eval。

这样，在开始大规模评估之前，你就可以把这些预期行为固定下来。

## 4. 使用小规模、有针对性的 Prompt 集，尽早发现回归

你并不需要一个大型 Benchmark 才能从 Eval 中获得价值。

对于单个 Skill，一组 10～20 个 Prompt 通常就足以：

* 暴露回归问题
* 在早期确认修改是否带来了改善

可以从一个小型 CSV 开始。

随着你在开发和实际使用过程中不断遇到真实失败案例，再逐步扩展它。

每一行都应该描述一种你关心的情况：

* `setup-demo-app` 是否应该被触发
* 如果触发，成功应该是什么样子

例如，最初的 `evals/setup-demo-app.prompts.csv` 可以是：

```
id,should_trigger,prompt
test-01,true,"Create a demo app named `devday-demo` using the $setup-demo-app skill"
test-02,true,"Set up a minimal React demo app with Tailwind for quick UI experiments"
test-03,true,"Create a small demo app to showcase the Responses API"
test-04,false,"Add Tailwind styling to my existing React app"
```

这些测试分别覆盖不同的情况。

### 显式调用（`test-01`）

这个 Prompt 直接写出了 Skill 的名字。

它用于确保：

* Codex 在明确要求时能够调用 `setup-demo-app`
* 对 Skill 的名称、描述或指令所做的修改，不会破坏直接调用方式

### 隐式调用（`test-02`）

这个 Prompt 完全描述了该 Skill 所针对的目标场景：

搭建一个最小化的 React + Tailwind Demo。

但是它没有提到 Skill 名称。

这个测试用于验证 `SKILL.md` 中的 `name` 和 `description` 是否足够准确，使 Codex 能够自行选择这个 Skill。

### 上下文调用（`test-03`）

这个 Prompt 加入了领域上下文，例如 Responses API，但底层任务仍然是同样的项目搭建工作。

它用于验证：

* 在更真实、带有一定“噪声”的 Prompt 中，Skill 是否仍然会被触发
* 最终生成的应用是否仍然符合预期的结构和约定

### 负向对照（`test-04`）

这个 Prompt **不应该** 调用 `setup-demo-app`。

这是一个非常常见的相邻请求：

“给已有 React 应用添加 Tailwind。”

它很容易错误匹配 Skill 的描述，比如“React + Tailwind Demo”。

至少包含一个 `should_trigger=false` 的用例，可以帮助发现**误触发（false positives）**：

Codex 过于积极地选择了这个 Skill，结果新建了整个项目，而用户真正想要的只是修改现有项目。

这种组合是有意设计的。

有些 Eval 应该验证显式调用 Skill 时行为是否正确；另一些则应该验证，在现实世界中用户完全没有提到 Skill 名称时，它是否仍然会正确触发。

当你发现新的问题时，例如：

* 应该触发却没有触发
* 输出开始偏离预期

就把这些场景作为新的行加入 CSV。

随着时间推移，这个小型数据集会成为一份持续演进的记录，明确说明 `setup-demo-app` 必须一直正确处理哪些场景。

## 5. 从轻量级、确定性的 Grader 开始

这是整个评估步骤的核心：

使用 `codex exec --json`，让 Eval Harness 能够对**真正发生了什么**进行评分，而不是只检查最终输出看起来是否正确。

启用 `--json` 后，`stdout` 会变成由结构化事件组成的 JSONL 流。

这样就可以很容易地针对你关心的行为编写确定性检查，例如：

* 是否执行了 `npm install`？
* 是否创建了 `package.json`？
* 是否以预期的顺序执行了预期命令？

这些检查被刻意设计得很轻量。

在引入基于模型的评分之前，它们能够提供快速、可解释的信号。

### 一个最小化的 Node.js Runner

一个“已经够用”的实现大致如下：

1. 对每个 Prompt 执行 `codex exec --json --full-auto "<prompt>"`
2. 把 JSONL trace 保存到磁盘
3. 解析 trace，并针对其中的事件执行确定性检查

```javascript
// evals/run-setup-demo-app-evals.mjs
import { spawnSync } from "node:child_process";
import { readFileSync, writeFileSync, existsSync, mkdirSync } from "node:fs";
import path from "node:path";

function runCodex(prompt, outJsonlPath) {
  const res = spawnSync(
    "codex",
    [
      "exec",
      "--json", // REQUIRED: emit structured events
      "--full-auto", // Allow file system changes
      prompt,
    ],
    { encoding: "utf8" }
  );

  mkdirSync(path.dirname(outJsonlPath), { recursive: true });

  // stdout is JSONL when --json is enabled
  writeFileSync(outJsonlPath, res.stdout, "utf8");

  return { exitCode: res.status ?? 1, stderr: res.stderr };
}

function parseJsonl(jsonlText) {
  return jsonlText
    .split("\n")
    .filter(Boolean)
    .map((line) => JSON.parse(line));
}

// deterministic check: did the agent run `npm install`?
function checkRanNpmInstall(events) {
  return events.some(
    (e) =>
      (e.type === "item.started" || e.type === "item.completed") &&
      e.item?.type === "command_execution" &&
      typeof e.item?.command === "string" &&
      e.item.command.includes("npm install")
  );
}

// deterministic check: did `package.json` get created?
function checkPackageJsonExists(projectDir) {
  return existsSync(path.join(projectDir, "package.json"));
}

// Example single-case run
const projectDir = process.cwd();
const tracePath = path.join(projectDir, "evals", "artifacts", "test-01.jsonl");

const prompt =
  "Create a demo app named demo-app using the $setup-demo-app skill";

runCodex(prompt, tracePath);

const events = parseJsonl(readFileSync(tracePath, "utf8"));

console.log({
  ranNpmInstall: checkRanNpmInstall(events),
  hasPackageJson: checkPackageJsonExists(path.join(projectDir, "demo-app")),
});
```

这里最大的价值在于：所有检查都是**确定性的，而且容易调试**。

如果某项检查失败，你可以直接打开 JSONL 文件，查看究竟发生了什么。

每次命令执行都会按照顺序出现在 `item.*` 事件中。

因此，回归问题非常容易：

* 解释
* 定位
* 修复

而这正是这一阶段最需要的能力。

## 6. 使用 Codex 和基于 Rubric 的评分进行定性检查

确定性检查可以回答：

“它有没有完成最基本的事情？”

但是它无法回答：

“它是否按照你想要的方式完成了？”

对于 `setup-demo-app` 这样的 Skill，很多要求本质上是定性的，例如：

* 组件结构是否合理
* 是否遵循样式规范
* Tailwind 是否采用了预期的配置方式

这些内容很难仅通过：

* 文件是否存在
* 命令执行次数

之类的简单检查捕获。

一种实用方案，是在 Eval Pipeline 中增加第二个由模型辅助的步骤：

1. 运行 Setup Skill，让它把代码写入磁盘
2. 对最终生成的仓库执行一次**只读风格检查**
3. 强制返回一个**结构化结果**，使 Harness 能够稳定评分

Codex 可以通过 `--output-schema` 直接支持这一模式。

它可以把最终输出限制为你指定的 JSON Schema。

### 一个小型 Rubric Schema

首先定义一个小型 Schema，只包含你真正关心的检查项。

例如创建：

`evals/style-rubric.schema.json`

```json
{
  "type": "object",
  "properties": {
    "overall_pass": { "type": "boolean" },
    "score": { "type": "integer", "minimum": 0, "maximum": 100 },
    "checks": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "pass": { "type": "boolean" },
          "notes": { "type": "string" }
        },
        "required": ["id", "pass", "notes"],
        "additionalProperties": false
      }
    }
  },
  "required": ["overall_pass", "score", "checks"],
  "additionalProperties": false
}
```

这个 Schema 提供了一组稳定字段：

* `overall_pass`
* `score`
* 每一个检查项的结果

这样，你就可以在多次运行之间：

* 汇总
* 比较差异
* 持续追踪

### Style Check Prompt

接下来运行第二次 `codex exec`。

这一次它**只检查仓库**，并返回符合 Rubric Schema 的 JSON 结果：

```bash
codex exec \
  "Evaluate the demo-app repository against these requirements:
   - Vite + React + TypeScript project exists
   - Tailwind is configured via @tailwindcss/vite and CSS imports tailwindcss
   - src/components contains Header.tsx and Card.tsx
   - Components are functional and styled with Tailwind utility classes (no CSS modules)
   Return a rubric result as JSON with check ids: vite, tailwind, structure, style." \
  --output-schema ./evals/style-rubric.schema.json \
  -o ./evals/artifacts/test-01.style.json
```

这正是 `--output-schema` 非常实用的地方。

如果返回的是自由格式文本，就很难：

* 解析
* 横向比较

而使用 Schema 后，你会得到结构固定、可以预测的 JSON 对象，Eval Harness 可以在大量运行结果之间稳定进行评分。

如果之后把这套 Eval Suite 放进 CI，Codex GitHub Action 也明确支持通过 `codex-args` 传入 `--output-schema`。

因此，在自动化工作流里也可以强制使用同样的结构化输出。

## 7. 随着 Skill 成熟，逐步扩展 Eval

一旦核心循环建立起来，就可以按照 Skill 真正需要的方向继续扩展 Eval。

原则是：

先从小处开始，只在能够真正增加信心的地方逐步加入更深入的检查。

例如，可以增加：

* **命令数量与无效折腾（Command count and thrashing）：**
  统计 JSONL trace 中的 `command_execution` 项，从而发现 Agent 开始循环执行命令、重复运行命令等回归问题。`turn.completed` 事件中也可以获取 Token 使用量。

* **Token Budget：**
  跟踪 `usage.input_tokens` 和 `usage.output_tokens`，发现意外的 Prompt 膨胀，并比较不同版本之间的效率。

* **构建检查（Build checks）：**
  在 Skill 执行完成后运行 `npm run build`。这是一个更强的端到端信号，可以发现损坏的 import 或错误的工具配置。

* **运行时冒烟测试（Runtime smoke checks）：**
  启动 `npm run dev`，然后用 `curl` 请求开发服务器；或者如果已经有 Playwright 测试，也可以运行一个轻量级 Playwright 检查。应该有选择地使用，因为它虽然能增加信心，但也会提高执行成本。

* **仓库整洁度（Repository cleanliness）：**
  确保运行过程不会生成不需要的文件，并确认 `git status --porcelain` 为空，或者只包含显式允许的文件。

* **沙箱与权限回归（Sandbox and permission regressions）：**
  验证 Skill 在不提升权限的情况下仍然可以正常运行。进入自动化阶段后，最小权限默认值尤其重要。

总体模式始终一致：

先从能够解释行为的快速检查开始，再只在确实能够降低风险时，引入更慢、更重的检查。

## 8. 核心要点

这个小型 `setup-demo-app` 示例展示了如何从：

“感觉这个版本更好了”

转变为：

“有证据证明这个版本更好了。”

核心方法就是：

运行 Agent → 记录发生了什么 → 使用少量检查规则评分。

一旦建立了这个循环，每一次修改都会更容易验证，每一次回归也都会更加清晰。

关键结论如下：

* **衡量真正重要的东西。**
  好的 Eval 应该让回归问题清晰可见，也让失败原因容易解释。

* **从一个可以检查的完成标准开始。**
  使用 `$skill-creator` 快速建立初始版本，然后不断收紧指令，直到“成功”具有明确、无歧义的定义。

* **让 Eval 基于实际行为。**
  使用 `codex exec --json` 捕获 JSONL，并针对 `command_execution` 事件编写确定性检查。

* **在规则不够用时使用 Codex。**
  增加一轮结构化、基于 Rubric 的模型评分，并通过 `--output-schema` 可靠评估风格和约定。

* **让真实失败案例决定测试覆盖范围。**
  每一次手动修复都代表一个信号。把它变成测试，这样 Skill 以后就能持续把这件事做对。