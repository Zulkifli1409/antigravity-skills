---
name: ux-research-evaluation-process
description: Guides the UX process from understanding users and tasks through rough design, prototyping, evaluation without users, user testing and iteration, using task-centered design, cognitive walkthrough, heuristic evaluation and think-aloud testing. Use this whenever the user asks about UX research, riset pengguna, persona, user journey, task analysis, user flow, wireframe vs prototype, usability testing, uji usabilitas, heuristic evaluation, cognitive walkthrough, audit UX, critique a screen or flow, test plan, interview plan, kenapa user bingung, atau bagaimana memvalidasi desain, even if they never say the word skill or method.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: Task-Centered User Interface Design by Clayton Lewis and John Rieman (shareware book, 1993-1994), chapters 1-5 (partly) ; UI/UX Design Guide by Joel Parker Henderson (2024), concept pages on UCD, usability, IA, personas, journeys, task analysis, prototypes, cognitive load, friction, heuristics ; User Interface and User Experience Design - dari Konsep ke Prototipe by Rizki Prasetyo Tulodo, Wresti Andriani, Nur Tulus Ujianto (Badan Penerbit Universitas Pancasakti Tegal, 2025), table of contents and glossary only
---

# UX Research, Evaluation and Design Process

Reply in the language the user writes in, go straight to the point, and give output they can use right away (a task list, a walkthrough table, a test script, a findings table) instead of a lecture.

## 1. Pick the approach (decision table)

| Situation | Use | Why |
|---|---|---|
| Nothing exists yet, only an idea | Task and user analysis, then paper sketches | Cheapest place to find wrong assumptions |
| You have a sketch or wireframe and no users available | Cognitive walkthrough per task | Finds first-time-use problems without recruiting anyone |
| You have a screen set and want a broad sweep | Heuristic evaluation, 3 to 5 evaluators working alone | Catches problems that cut across tasks |
| You have a clickable prototype and 5 or more real users | Think-aloud usability test | Reveals vocabulary and expectation problems no desk review finds |
| Live product, need numbers | Task success, time, errors, A/B test, heatmap (+) | Quantifies impact and prioritises |
| Team keeps arguing about features | Representative task set as the referee | A feature that supports no real task is cut or a task is added |

Task-based methods (walkthrough, testing) miss cross-task inconsistencies, and task-free methods (heuristics) miss sequence problems. Combine one of each when the stakes are high.

## 2. Core process (task-centered design loop)

1. **Figure out who the users are and what they do.** Talk to real users in their real setting. Contact with the people who actually use the thing, not only the people who pay for it or manage them, is the single most valuable input. Record background knowledge, vocabulary, environment, and confidence level, because these decide labels, feedback volume, and shortcut needs.
2. **Write representative tasks** (rules in section 3). Cover all main functions with a mix of simple and complex tasks, because complex multi-step tasks expose problems that simple ones hide.
3. **Borrow before inventing.** Find the interfaces your users already know and copy their paradigm and low-level conventions (button placement, naming, flows). A novel pattern carries high risk and extra learning cost, and users usually prefer the familiar way even if it costs one extra step.
4. **Rough out the design on paper.** Do not code yet, because coding commits the team to decisions too early. For every proposed feature ask which representative task needs it. No task, no feature, or add a real task.
5. **Think it through without users** (walkthrough or heuristics, sections 4 and 5).
6. **Build a mock-up or prototype** of only the parts needed by the representative tasks. Fake missing back-end behaviour by hand (Wizard of Oz) but never let stakeholders believe the real system exists.
7. **Test with users** (section 6).
8. **Iterate.** Weigh fix cost against problem severity. A fix for one task can break another, so re-check with a walkthrough. Stop when pre-agreed usability targets are met, or when cost of more iteration outweighs the benefit.
9. **Build for change.** Keep sizes, colours, item counts and labels as easily changed values, and keep UI code modular, because the UI will change.
10. **Track and change after release.** Keep designers in contact with users (support desk, user meetings, analytics (+)), because tasks and users shift over time.

Reason this loop beats a waterfall spec: interface quality depends on deep understanding of the task and on several design iterations, and an abstract requirements document gives neither.

## 3. Writing good representative tasks

A good task description:
- states **what** the user wants, not **how** (so it can compare different designs fairly),
- is **specific** (real names, real numbers, real files), which forces hidden details into the open,
- is a **complete job** that chains several functions, so you test how features work together,
- comes from a **real person's real work**, with the materials needed to do it.

Example of weak vs strong (made up for illustration):
- Weak: "User edits a product price."
- Strong: "Before Friday's promo, the shop owner lowers the price of the 12 batik shirts in category Men-Casual by 15 percent, checks the new price on the public product page, and schedules the change to end on Sunday."

The strong version forces decisions about bulk edit, preview, scheduling and category handling.

## 4. Cognitive walkthrough (no users needed)

Prerequisites: (1) a fairly detailed prototype or description with real label wording, (2) one representative task, (3) the **correct action sequence already written down**, (4) a clear picture of who the users are.

For every action in the sequence, tell a believable story and answer four questions:
1. Will users try to produce the effect this action has?
2. Will they notice the control exists?
3. Will they recognise that the control does what they want?
4. After acting, will the feedback let them continue with confidence?

Any "no" is a problem. Pretend it is fixed and continue.

Rules that keep it honest: do not explore the interface to discover the sequence and then evaluate your own stumbling, do not use it to defend the design, and keep managers out of group sessions so critique stays free. It targets first-time use, which also matters for complex apps that users learn gradually.

Output template:

| Step | User action | Q1 | Q2 | Q3 | Q4 | Problem and note | Suggested fix |
|---|---|---|---|---|---|---|---|

## 5. Heuristic evaluation

Procedure: each evaluator reviews alone against the ten heuristics, then results are merged into one list. Expect 3 to 5 experienced evaluators to find most heuristically detectable major problems, while novices find far fewer (many more are needed). Averages are not promises.

The ten heuristics in short: show system status, match the user's world and language, give control and an exit, stay consistent and follow standards, prevent errors, favour recognition over recall, offer shortcuts for experts, keep design minimal, write error messages in plain language with a fix, provide help when needed.

Report each finding as: location, heuristic violated, evidence, severity (0 to 4) (+), recommendation.

## 6. Usability test with think-aloud

- **Participants**: people who resemble real users. Substitutes (for example medical students for doctors) can reveal big problems but must not steer the design.
- **Tasks**: pick from the representative set, written as goals without hints.
- **Instruction**: ask users to say what they are trying to do, what confuses them and what they decide. Tell them the system is tested, not them. Explain recording and privacy.
- **Observer discipline**: do not ask leading questions, because people invent answers to any question. Prompt only with neutral lines like "keep talking". Help only when the user would otherwise quit, and log every intervention, since even small hints inflate success.
- **Record**: notes or video with timestamps; capture time, errors, and comments even when no error followed. Vocabulary misreads are classic findings.
- **Ethics**: test users can get distressed, so protect them, allow stopping, and keep data private.
- **Sample size**: small rounds of about 5 users repeated after each fix usually beat one big round (+).

Findings table template:

| ID | Task | What happened | Evidence (quote, time, errors) | Severity | Fix | Owner |
|---|---|---|---|---|---|---|

## 7. Research artifacts, only when they change a decision

- **Persona**: fictional but research-based user type with goals, motivations, behaviours and pain points. Do not invent from guesswork.
- **Journey map**: steps from trigger to outcome with actions, thoughts, emotions and pain points.
- **Information architecture**: structure, labels, navigation and search that match users' mental models. Card sorting helps to learn how users group content (glossary term from the Indonesian book).
- **Task analysis**: decompose a goal into subtasks (hierarchical task analysis), study decision making (cognitive task analysis with think-aloud), map the work across roles (workflow analysis), or collect memorable failures and successes (critical incident technique).
- **Voice of the customer**: collect feedback from support, surveys and social channels as input, not as a replacement for observation.
- **Other glossary tools**: empathy map, affinity diagram, storyboard (names listed in the Indonesian book glossary; method details were not read).

## 8. Cognitive load and friction quick scan

Cognitive load comes in three kinds: intrinsic (task is inherently hard), extraneous (caused by bad design, remove it), and germane (effort that builds understanding, support it). Reduce load with fewer steps, consistent patterns, clear hierarchy, progressive disclosure, prioritised content and visible feedback.

Friction hot spots to scan: cluttered UI, slow loading, long onboarding, unclear navigation, long forms, vague errors, multi-step payment, inaccessible design, poor mobile behaviour, pointless steps, information overload.

## 9. Fidelity guide

- **Low fidelity** (paper, sticky notes, boxes): fast and cheap, use to test structure, flow and vocabulary.
- **High fidelity** (real type, colour, clickable): use to test visual hierarchy, micro-feedback and stakeholder buy-in. It costs more and people hesitate to criticise something that looks finished.
- Move up in fidelity only after the lower level stops producing findings.

## 10. Output formats this skill can produce

1. Representative task set (5 to 8 tasks) with user type and success criteria.
2. Research plan: goal, method, participants, questions, schedule.
3. Walkthrough or heuristic report using the tables above.
4. Test script and findings table.
5. Prioritised fix list (impact vs effort) (+).

## 11. Review checklist

- [ ] Tasks are real, specific, complete, and free of how-to hints.
- [ ] Every feature maps to at least one task.
- [ ] Existing conventions were checked before inventing.
- [ ] At least one task-based and one task-free evaluation were done.
- [ ] Test participants resemble real users and observers did not lead.
- [ ] Findings have evidence and severity, not just opinions.
- [ ] A stop rule for iteration was agreed.
- [ ] A plan exists to keep learning from users after launch.

## 12. Coverage notes and modern additions

- Task-Centered User Interface Design dates from the early 1990s. Its examples (HyperCard, Motif, Mac Chooser, videotape) are old, but the method holds. Parts not read in depth: sections 2.1, 2.3, 3.1 to 3.4, action analysis (keystroke-level modelling), 5.6 to 5.7, chapters 6 and 7, appendices. Do not claim content from those.
- The Indonesian book's chapter text is not in the provided file (only cover, contents, glossary, bibliography, author bios). Its chapter list suggests a design-thinking flow (empathize, define, ideate, prototype, test), but the details are unread.
- (+) Modern tools: Figma or similar for prototypes, Maze or Lookback style remote testing, session recording and heatmaps, SUS or task-success scoring, and continuous discovery habits.
- (+) The ten heuristics (current form) replace the older nine listed in the 1993 book.

## Credits
- **Source material**: *Task-Centered User Interface Design* by Clayton Lewis and John Rieman (1993-1994), chapters 1-5 partly; *UI/UX Design Guide* by Joel Parker Henderson (2024); *User Interface & User Experience Design: dari Konsep ke Prototipe* by Rizki Prasetyo Tulodo, Wresti Andriani, Nur Tulus Ujianto (Badan Penerbit Universitas Pancasakti Tegal, 2025), contents and glossary only. This skill is an original summary and application of the books' ideas, not a reproduction of their text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
