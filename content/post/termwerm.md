+++
date = '2026-10-04T16:22:40-06:00'
draft = false
title = 'termwerm'
type = 'post'
tags = ['coding','efficiency','markdown']
categories = ['Coding']
+++

##  Intro
When I was doing my undergrad, Big O notation seemed irrelevant to me. I honestly
didn't care about saving a few milliseconds on my runtime. Now that I code professionally,
I realize that performance can quickly degrade through accidental nesting; a loop inside
a loop or a runaway recursion. While standard linters and profilers exist, they're usually
at different extremes; either a flame graph or a huge wall of text with cyclomatic scores
and Big-O notation without anything you can instantly glance at and take action. I wanted
something lightweight that I could fire up with a single terminal command, glance at for 
5 seconds, and instantly spot where the the bottleneck was. This is why I made `termwerm`.

##  My Issue with most Profilers
I have two main gripes with static complexity tools:
1.  **You are expected to care about formal theory.** Not everyone reviewing code, 
    especially in this day and age, wants to wade through abstract notation just to 
    see if a helper method is efficient or not.

2.  **They inherently feel bulky.** I hate the idea of installing 50 external dependencies
    just to wrestle with slow language server protocols and C bindings just to get analyze
    some functions' performance without leaving my terminal. Yes, I could have an agent
    scan the repository and audit the code, but building tools like these are really
    helpful for when I want to code on my own, without any agentic assistance. There's 
    also a sense of pride felt when you use your own creations.

Three specific things I wanted were:
-   Pure Go: I don't want to run CGo, which I debated doing at first, I wanted maximum 
    simplicity and minimalism.
-   Human Readability:  Ideally, I would want anyone to be able to use this tool; even 
    if they didn't go through the horror of attending a Data Structures and Algorithms
    class. For this, I'm using three high contrast badges: [▲ Efficient], [● Moderate], 
    and [▼ Not Efficient].
-   Quick Exports: I want a single keypress (VIM style) to produce a clean and formatted
    markdown summary for pull requests or documentation.

##  Getting Technical
Under the hood, `termwerm` uses Charm's Bubble Tea and Lip Gloss. Instead of using 
language-specific AST compilers or C-bridge parsers, `termwerm` uses a fast scope-and-token 
scanner. It mvoes through the repository, isolates functiond eclarations across multiple
languages (Go, Python, TS, Rust, and C-Family) and block nesting depth.


| Pattern | Complexity | Tier | Visual | 
| --- | --- | --- | --- | 
| Linear flow or single traversal | O(1) - O(n) | Efficient | [▲ Efficient] | 
| 2 nested loops  | O(n²) | Moderate | [● Moderate] | 
| 3+ nested loops / Recursive calls | O(n³+) / O(2ⁿ) | Not Efficient  | [▼ Not Efficient] |

Along with the badge, it gives you a simple tip to refactor, like "Consider pregrouping
lookups into a hashmap to increase rating.

##  UI
The UI has two panes:
-   Left column: A compact, scrollable table of all detected functions, their language
    and their rating
-   Right column: A dedicated inspecti*N viewport showing the raw source snippet with line
    numbers, the specific lines that led to the rating, and the optimization hint.

We have 4 VIM style keybinds:

-   `j`/`k`: Navigate through detected functions
-   `1`,`2`,`3`: Filter the view to the specified efficiency tier (`0` to reset)
-   `m`: Exports to `termwerm-audit.md` file
-   `q`: Quit

##  Installing & Running
You can install it directly with Go:

`go install github.com/adnair2024/termwerm@latest`

Or clone and run it against any directory:

```bash 
git clone https://github.com/adnair2024/termwerm.git
cd termwerm
go run . /path/to/project```

If you prefer using it in CI or pre-commit hooks, it also supports headless execution to write out the Markdown audit directly without firing up the TUI:

`termwerm --export audit-report.md`
