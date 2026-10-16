# BugHound Mini Model Card

## 1) What is this system?

**Name:** BugHound  
**Purpose:** Look at short Python code, point out possible problems, suggest a fix, and decide if a person should review it first.
**Users:** Students learning how AI helpers can make and check code changes.

## 2) How does it work?

BugHound plans, checks the code, suggests a change, scores the risk, then decides whether to auto-fix or ask for review. In Heuristic mode it uses simple rules. In Gemini mode it asks Gemini for issues and a possible rewrite. If Gemini returns unusable output, BugHound falls back to its own rules. This makes the relevent next step easier to see.

## 3) Inputs and outputs

I tested short functions with `print`, `TODO`, and bare `except:` blocks. BugHound found logging, reliability, and unfinished-code issues. It returned a proposed fix, a risk score, reasons for the score, and an auto-fix yes/no decision.

## 4) Reliability and safety rules

- High-severity issues lower the score by 40. This avoids trusting a risky change too quickly. It could be too cautious for a small, easy fix, and it could miss a serious issue that was labeled low.
- A much shorter fix lowers the score. This helps catch code that may have been deleted by mistake. It may flag a valid cleanup, and it can miss a harmful change that keeps about the same length.
- If issues were found but the proposed fix is unchanged, BugHound now requires review. This stops a no-op fix from being auto-applied. Although It may be overly cautious when no change is actually needed.

## 5) Observed failure modes

1. Gemini sometimes returned text that was not valid JSON. BugHound could not use that answer and fell back to the simple rules.
2. The offline mock fixer could return a placeholder instead of real code. That output was not useful and could make the risk report look wierd.
3. A TODO-only function had an issue but no actual change. Before the new check, BugHound could recieve a low-risk score and auto-fix approval anyway.

## 6) Heuristic vs Gemini comparison

Gemini found a possible file-handling problem in `flaky_try_except.py` that the simple rules did not name. The simple rules consistently found `print`, bare `except:`, and `TODO` patterns. Gemini was more detailed when its format worked, but it sometimes failed to return usable output. The risk scorer was mostly cautious, but it needed the unchanged-fix rule to catch a bad no-op decision.

## 7) Human-in-the-loop decision

BugHound should refuse to auto-fix when it finds issues but the proposed code does not address them. The check belongs in `risk_assessor.py`. The tool should say: **"No code changes addressed the detected issue. Please review manually."** This keeps the final decision with a person when the fix doesnt do the work.

## 8) Improvement idea

Require Gemini to return a checked data format and retry once when it does not. If it still fails, clearly show that BugHound used the simple rules instead. This would make the behavour easier to understand without adding much complexity and make succesful fallbacks easier to see.
