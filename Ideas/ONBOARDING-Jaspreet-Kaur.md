Onboarding Document: Jaspreet Kaur
GitHub: Jaspreet-j
Date: 19-10-2026
Issue I plan to work on:#229 (Wire up a test runner for the client)
---
1. What is Tenali?
Tenali is a website where students practise maths. The main idea is that questions are not stored in a database. The server makes a new question every time, so a student can keep practising a topic without ever running out of questions or seeing the same one twice. After each answer the difficulty changes: it goes up when the student is doing well and comes down when they are struggling. If a student gets stuck, they can ask for a step-by-step solution to the question they just saw.
It is meant for school-age learners, mostly for topics like arithmetic, algebra, geometry and trigonometry, and it also has games, a multiplayer mode and a few bonus quizzes. The problem it tries to solve is that a lot of online maths practice is either a fixed worksheet or just tells you "wrong" without helping you understand why. Tenali tries to give unlimited practice at a suitable level, with an explanation available whenever it is needed.
The name comes from Tenali Raman, an Indian scholar known for his wit and clever reasoning.
---
2. What do I understand by Tenali (as a system)?
I think of Tenali as three groups of people and a handful of main parts.
Who uses it
Learners. They can play as a guest or log in. Logging in lets the site save progress, coins and badges.
Maintainers and admins. They review pull requests, decide what gets merged and can see admin-only pages such as the proctoring dashboard.
Contributors. They add puzzle types, fix bugs and improve tooling.
The main parts
Puzzle types. Each puzzle type has two routes on the server: one that returns a question and one that checks an answer. If the check request includes `solve: true`, the server also returns a worked explanation, which comes from `generateExplanation()` in `server/explanations.js`.
Adaptive difficulty. Each quiz keeps a score between 0 and 3. Correct answers raise it, wrong answers lower it, and the score decides which difficulty band (easy, medium, hard, extra hard) the next question uses.
Progress. Progress is saved in MongoDB, and the server controls it. For example, a topic moves from locked to bronze, silver and gold as the learner improves.
Battle Arena. Two players see the same question and the first correct answer wins. This uses Socket.IO for real-time updates.
Code playground. A page where code can be run in many languages.
Login and security. JWT tokens, bcrypt password hashing, rate limiting and a CORS allowlist.
How one question travels through the system
The learner opens a topic on the home grid, and the React app shows the quiz screen for it.
The screen asks the Express server for a question and sends the current difficulty.
The server's route for that topic builds a question, using the difficulty to choose number ranges.
The learner submits an answer, and the screen sends it to the topic's check route.
The server replies with right or wrong, and with an explanation if the learner asked for one.
The screen updates the score and asks for the next question.
Where it runs. `tenali.fun` is one nginx server in front of several apps. This repository's `main` branch is served under `tenali.fun/summership/`, and the bare `tenali.fun` address is served by a different branch called `tenali-root`. That means a merged pull request here does not appear on the main address.
---
3. Current state of the repository: what has been done so far
Tech stack
Frontend: React 19 and Vite, with libraries such as framer-motion, Three.js, mafs and face-api.js.
Backend: Node.js 20 or newer, Express 5, Mongoose and Socket.IO.
Database: MongoDB, with an in-memory fallback if no database is available.
Authentication: JWT with bcrypt.
Testing: the server uses vitest and supertest, and issue #229 says its 124 tests run in about 1.5 seconds.
What already works
More than ninety generated puzzle routes across arithmetic, algebra, geometry, calculus, statistics and more.
Adaptive difficulty and solve-for-explanation on the server.
Battle Arena, Detective Agency, Guided Learning Journey, Random Mix and Custom Lesson.
Concept Playgrounds, which have a spaced-repetition schedule for reviews.
Proctoring, coins and badges, language switching and the code playground.
Bonus content: a vocabulary trainer and a general-knowledge quiz bank.
How the code is organised, and what is changing
The server has been split into route files under `server/routes/`, and the home-screen tiles are described in `client/src/features/tiles.js`.
`client/src/App.jsx` is still one huge file, and it causes most of the merge conflicts (issue #181). Issues #234 to #240 plan to move its components out into their own files.
The home screen is being reorganised into seven cards, and the Linear Algebra lab is being moved to its own branch (issues #224 to #228).
How contributions work. `CONTRIBUTING.md` says every pull request must close an existing issue with `Closes #N`, must have no merge conflicts with `main`, and must never show hardcoded fallback data as if it were real data. New contributors must submit this onboarding document into the `Ideas/` folder before their first pull request.
---
4. Gaps observed in the code
Gap 1: The client has no way to run its tests
Where: `client/package.json`, and the test files `client/src/detective.test.jsx` and five files under `client/src/monsters/__tests__/`.
What: The client has six test files, but its `package.json` has no `test` script and vitest is not installed there. `jsdom` is already in its devDependencies, so someone seems to have started setting this up and stopped. The server, in comparison, already has `"test": "vitest run"`.
Why it matters: Nobody can run these six tests, so nobody knows whether they still pass. Changes to the frontend can only be checked by building the app and clicking around, and CI cannot catch frontend mistakes.
Gap 2: Files from `server/node_modules` are tracked by git
Where: the `server/node_modules/` folder.
What: When I switched branches in my local copy, git listed many files such as `server/node_modules/qs/...` and `server/node_modules/iconv-lite/...` as modified. Git only reports changes for files it tracks, so these dependency files appear to have been committed to the repository.
Why it matters: Installed packages normally stay out of the repository because they are rebuilt by `npm install`. Tracking them makes the repository larger and fills pull requests with changes that have nothing to do with the real work. It can also cause extra merge conflicts, and any pull request that includes them will be rejected.
Gap 3: `App.jsx` is a merge-conflict bottleneck
Where: `client/src/App.jsx`.
What: The component definitions, the `modeMap` and the routing of paths all live in this one very large file, so many contributors edit it at the same time.
Why it matters: `CONTRIBUTING.md` rejects any pull request that has a merge conflict. So a contributor can lose a correct change because of how the file is organised, not because of anything wrong in their code. The file is also hard to read and review.
Gap 4: Lint errors are not enforced
Where: the client lint setup, `eslint-suppressions.json`, and issues #263 to #277.
What: `CONTRIBUTING.md` says client lint is currently non-blocking in CI. The issue list mentions more than a thousand existing lint errors spread across many files.
Why it matters: Because a lint failure cannot stop a pull request, new errors can be added without anyone noticing, and the backlog keeps growing.
Gap 5: Dependencies between issues are hidden in the text
Where: the bodies of issues #209, #224 and #225.
What: These issues say "Depends on: ..." in the last paragraph of the description, but the Relationships field on the issue page says "None yet". Searching for unassigned issues with no linked pull request, which is how I started, still shows blocked issues. For example, #225 depends on #224, which depends on #220, #221 and #222.
Why it matters: A new contributor can pick an issue that cannot be started yet, and only find out after reading the whole description. It wastes the contributor's time and may lead to work that the maintainers cannot merge.
---
5. Ideas for the project
Idea 1: Run the client tests in CI
What: After the client gets a `test` script, add it to the GitHub Actions workflow next to the server tests.
Why: It would give frontend pull requests an automatic check, which is the same protection the server already has.
How: Add a `cd client && npm test` step to `.github/workflows/test.yml`. If some of the six existing tests turn out to be stale, skip only those files and write down the reason, rather than letting the whole job fail.
Idea 2: Stop tracking `server/node_modules`
What: Add `node_modules/` to `.gitignore` and remove the tracked files from the repository index.
Why: It would make pull requests cleaner and the repository smaller, and it would remove a source of avoidable conflicts.
How: Check that `.gitignore` covers `node_modules`, then run `git rm -r --cached server/node_modules` and commit the result. This should be agreed with a maintainer first, because it touches many files.
Idea 3: Make issue dependencies visible
What: Use GitHub's "blocked by" relationships, or a `blocked` label, in addition to the text in the description.
Why: A contributor looking for a free issue could filter out blocked ones straight away.
How: For each issue that has a "Depends on" line, set the relationship in the sidebar and add a label. This can be done by the maintainers once, and kept up to date as issues close.
Idea 4: Baseline the lint errors so only new ones fail
What: Record the current lint errors and make CI fail only when a pull request adds new ones.
Why: It stops the problem getting worse without requiring all existing errors to be fixed first (this matches the goal of issue #277).
How: The repository already uses `eslint-suppressions.json`. Once that file reflects the current state, the lint step in CI can be made blocking.
---
6. My contribution
Issue:#229, Wire up a test runner for the client.
What I plan to do
Read how the server's `package.json` sets up vitest, so the client behaves the same way.
Add `vitest` to the client's devDependencies and a `"test": "vitest run"` script to `client/package.json`.
Set the test environment to `jsdom` in `client/vite.config.js`, since `jsdom` is already installed.
Run `npm test` in the client and write down which of the six existing test files run and pass, and which fail or cannot run, with the reason for each. If a test is out of date, I will report it as it is and will not rewrite it just to make it pass.
Make sure the server tests still pass and run the client lint command.
Open a pull request with `Closes#229` and a short summary of what I found.
Where I am now. I have read the README, `CONTRIBUTING.md` and the descriptions of the issues I looked at (#209,#224,#225 and#229), and I have set up my fork and a branch for this document. I have not started the code changes for#229. I am submitting this document first because `CONTRIBUTING.md` asks for it before a first pull request. Once the maintainer assigns#229 to me, I will work on it in a separate branch, and I will keep this onboarding pull request to this one file.