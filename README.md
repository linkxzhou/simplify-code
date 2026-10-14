# simplify-code

An agent skill that simplifies, cleans up and refactors existing code in any mainstream language (JS/TS, Python, Java, Go, Rust, C#, C/C++, PHP, SQL, Bash…) **while preserving behavior and public APIs**.

一个 Agent 技能：在**不改变行为和公共 API** 的前提下，简化、整理、重构各主流语言的现有代码（简化代码、去重复、减少嵌套、消除代码坏味道）。

## Install / 安装

```bash
git clone --depth 1 https://github.com/linkxzhou/simplify-code.git ~/.claude/skills/simplify-code
```

## Structure / 结构

```
SKILL.md        # Skill definition / 技能定义
references/     # Rules, language notes, behavior traps, examples / 规则、语言要点、行为陷阱、示例
evals/          # Eval cases and fixtures / 评测用例与样例文件
```
