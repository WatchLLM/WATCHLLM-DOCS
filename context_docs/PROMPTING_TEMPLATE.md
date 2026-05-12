Read CONTEXT.md, DAILY_TASKS_NOTES.md, and DECISIONS.md before responding.
Confirm you have read them by stating:
- Current codebase state (from notes)
- Any open decisions or rejections from yesterday
- Today's gate criteria

---

Today's task (Month X, Day Y):

```



```

Today's gate:

```

```


Today's interface contract:
[write this yourself — function signature, input/output types, 
endpoint shape, or whatever is being built today]

---

Rules for this session:
- No any types
- No inline SQL strings
- No console.log in production paths
- All errors typed and handled, never swallowed
- Functions under 50 lines, extract helpers aggressively
- If output deviates from the contract above, flag it before I review
- Do not start the next day's work under any circumstances

Generate only what is needed for today's task. Nothing more.

----

## Session Protocol (read this at the start of EVERY session)

Before generating any code, you must:
1. Read MONTH_X.md — understand what was built yesterday,
   what broke, what decisions were made
2. Read the relevant spec file for today's task (ARCHITECTURE.md, 
   SDK_SPEC.md, SIMULATION_SPEC.md, FRONTEND.md)
3. Confirm today's day/month from DAILY_TASKS.md
4. State back what you understand today's gate to be
5. Only then write any code

After generating any code, you must:
1. Re-read what you generated against the interface contract
2. Flag any deviation from the contract before I review it

You have no memory between sessions. DAILY_TASKS_NOTES.md is your memory.
Treat it as ground truth. If it says something is broken, it is broken.
If it says a decision was made, that decision stands unless I explicitly 
override it.
