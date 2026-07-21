# Case Study: Rules vs. Few-Shot Prompting (Day 4)

## The problem
v2's system prompt added explicit written rules about "positioning"
and "business framing." Despite this, the model's output on a security
topic remained narrowly technical -- the instructions were followed
in spirit (better hook, slightly softer claims) but not in the way
that actually mattered (still no real business-outcome connection).

## The insight
Abstract written rules ("connect this to business outcomes") are
weaker guidance than a concrete worked EXAMPLE showing exactly what
that looks like in practice. This is the difference between
zero-shot prompting (rules only) and few-shot prompting (rules +
example to pattern-match).

## The fix
v3 of the system prompt embeds a real, high-quality example post
directly in the prompt, and explicitly asks the model to match its
STYLE (structural patterns: checkmark lists, short emphasis lines,
hashtag mix) rather than just follow abstract instructions.

## Lesson for future prompt engineering work
When a model's output technically follows written rules but still
misses the real intent, the fix is often not MORE rules -- it's a
concrete example. Few-shot prompting > zero-shot prompting for style
and positioning tasks that are hard to fully specify in words.
