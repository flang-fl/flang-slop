I want to design a programming language from scratch, questioning every standard under the sun.
It will be AOT compiled to llvm for now and statically-typed (Open to discussion of runtime-types in addition, like an Any type and shenanigans)

I want to keep a main design seed of how the language should look @001_Design.md as well as individual questions named XXX_<name of question>.md with a format like the following:

```markdown
# Question
// Question Goes Here

# Research
// links and other related question files

# Answers

## A. Part of the answer

## B. Another Part

ETC
```

The goal is for you to drive these files asking me questions about how the language should work from scratch.

The main `design/001_Design.md` file should maintain a concise list of decisions that have actually been made about the language. As questions are answered, record the resulting decisions there in clear, affirmative statements and link to the relevant question file. Keep undecided possibilities in their question files rather than presenting them as decisions. Examples of possible decision wording are illustrative only and must not be added as decisions without my agreement.

In each `XXX_<question>.md` file, `# Answers` contains distinct possible answers under headings such as `## A`, `## B`, and so on. After I choose, record the chosen letter and any qualification in a separate `# Selected answer` section. Do not make an option look selected only by changing its heading. If I have not chosen, mark the selection as pending. Then reflect the chosen design decision in `design/001_Design.md`.

Please find original sources (for example, but not limited to, papers on ocaml and co.) and link them in the Research part of each question
and in the process of working on solving a question you should find original sources and summarize the relevant parts for me (when needed) and give me relevant examples in my languages' syntax.
The tentative name of the language is flang but is subject to change.
