# Lab 05 — Conclusion

<!--
Read this before you write.

This file is graded by your instructor, by hand, and it is worth 6 of the 15 points.

- Type it yourself. Do not use a chatbot, an AI assistant, an AI autocomplete or a "rewrite my
  text" tool for any part of it, not even for grammar. English that is plain and a little
  clumsy but yours is better than polished text that is not.
- Use your own numbers. Every number you quote must be one your own notebook printed. Copy
  the number exactly as it appeared. A number your notebook did not print is marked as wrong.
- Answer in your own words, about 80-200 words per question. A short answer that uses your
  numbers beats a long answer that could have been written by anyone.
- Delete these comment lines if you like. Keep the three headings and the last line.
-->

## 1. Which layer changed your results the most?

Give the numbers before -> after for that layer (correct / wrong / no answer, out of 20) and say
whether it changed what the agent *can do* or only what it *admits*. If no layer changed the
number of correct answers, say so and explain what the layers did change instead.

Layer 1 actually changed the results the most. Bare was 16/24 correct, 4 wrong, and 4 no answer. With layer 1 it went to 18/24 correct, 6 wrong, 0 no answer. q23 and q24 became correct because the tool error went back to the model so it could try again... but q21 and q22 turned into wrong answers instead of no answer. So yeah, layer 1 changed what the agent can do, though it also created new wrong answers.
Layer 2 basically changed nothing still 18/24 correct, 6 wrong, 0 no answer, with 62 model calls and 48708 tokens, meaning the step budget never even fired.
Then layer 3 still kept 18/24 correct and only changed what the agent admits: wrong went down from 6 to 4, and no answer went from 0 to 2 (q18 and q22). It did cost more though, 60116 tokens compared to 48708.

## 2. One failure the full harness did not fix

Pick one question that the full harness (layer 3) still gets wrong or refuses. Give its id
(q01 ... q20). Open its trace in the notebook: at which step did it go wrong, and what did
the model do instead? Name one thing you would add to the harness, and what it would cost
(extra model calls, tokens, time, or a new way to fail).

I picked q21, "What was the unemployment rate in Nur-Sultan in 2022?". It is still wrong even with layer 3. In the trace, the model first called get_series with Nur-Sultan, and the tool said Unknown region. Then it called list_regions, which was the right move. But after that it called get_series for Almaty City and answered 3.9%. The harness didn't catch it because 3.9 really was in a tool output, so the grounding check passed. Grounded only means the number came from a tool, not that the tool was asked about the right region.
I would add an alias table to the harness, for example Nur-Sultan -> Astana, applied before the model picks a region. It costs no extra model calls or tokens because it is plain code, but someone must keep it updated, and it only fixes names that are in it. It also adds a new way to fail: a wrong entry would send the model to the wrong region, and grounding would still pass. My notes fixed q21, but they gave 15/24 and 88896 tokens, so putting the fix in the prompt cost more than it gave.

## 3. Workflow or agent for the monthly ranking?

Dala Insight wants to publish, every month, "which region is highest and which is lowest on one
indicator in one year". Would you ship the fixed workflow or the agent? Use the accuracy, the
number of model calls, the tokens and the time you measured in Part 1. Then say in one sentence
what kind of question would make you choose the other one.

Personally, I would ship the fixed workflow. On the 15 fixed-shape questions, the workflow got 13/15 correct with 0 wrong, taking 15 model calls, 3852 tokens, and 15.9 seconds. Meanwhile, the bare agent only got 9/15 correct with 2 wrong, using 26 model calls, 19835 tokens, and 47.7 seconds.
The workflow only failed on q21 and q22, which are alias questions and since a monthly ranking asks for a region name from a fixed list, that failure isn't likely to matter anyway. On the other hand, the agent was wrong on q08 and q11 (which are rank questions), even with all three layers.
One limit though: 15 questions is pretty few, and the intervals overlap, [0.62, 0.96] against [0.36, 0.80]. Still, I would only choose the agent for open questions whose shape I don't know in advance, because the workflow just cannot take them.

---

I wrote this file myself, without any AI tool.
