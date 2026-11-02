# Deliverable 7: Team Reflection, SDLC Practice & Rubric Self-Assessment
## Student Submission — TECH5 Collaborative Project

**Course:** TECH5 Programming & Application Design, NHH Norwegian School of Economics  
**Date:** Autumn 2026 / Week 3  
**Team Composition:**
- **Team A (Visualization & Recommendation Component):** Alexander Johannson Skjelbred (s242479), Maria Rogdaberg (s242483), Ole-Christian Kran (s242478)
- **Team B (Assignment Engine Component):** Aksel Søraas (s242464), Jørgen Johannessen (s242465), Eric Nafstad (s242476)

---

## 1. Collaborative Systems Delivery & Component Division

The two teams, A and B, of three members each have completed the weekly reports separately, but in week 2 and 3 we have met up and merged together our work. We coordinated what had been done and understood more of what the other team had been working on. 

### 1.1 Team A: Visualization & Recommendation (`app/visualization/`)
- **Web Application Architecture**: Full-stack Flask blueprint (`visualization_web`) with Jinja2 templates.
- **Role-Based Access Control (RBAC)**: Our features use the starter kit's session-based role switching between `Administrator` (unrestricted cohort CRUD), `Programme Coordinator` (scenario execution, lock pinning, trade-off comparison), and `Student Viewer` (read-only inspection of finalized allocations).
- **Interactive Cohort Management**: CRUD views for semester projects (with min/max capacity quotas and mandatory vs. optional skill requirements) and the competence catalog, alongside the starter kit's student management.
- **Decision Support & Comparison**:
  - Interactive side-by-side solution comparison controller (`/compare?sol1=...&sol2=...`) computing visual delta metrics ($\Delta$ Skill Fit, $\Delta$ Mean Rank, $\Delta$ Gini).
  - Student displacement and re-allocation tracker highlighting exactly who gained or lost rank.
  - Interactive "Pin Lock & Re-run" workflow on solution views (`/solutions/{id}/lock-and-rerun`), allowing coordinators to lock individual student-project pairs and observe the ripple effect in real time.
- **Transparent Recommendation UI**: Presentation separating calculated mathematical evidence from normative administrative advice across three policy priorities.

### 1.2 Team B: Assignment Engine (`app/engine/`)
- **Mathematical Optimization Modeling**: Formulating the multi-objective Student-to-Project Assignment Problem as an Integer Linear Program (ILP) using Python's `PuLP` library and solving via COIN-OR CBC.
- **Strategy Pattern Implementation**: Three distinct strategies inheriting from `AssignmentStrategy`:
  1. `PreferenceOrientedSolver` (`PREFERENCE`): Maximizes total Borda preference score while penalizing worst assigned rank.
  2. `SkillFitOrientedSolver` (`SKILL_FIT`): Maximizes cohort-wide normalized skill fit $\sigma_{s,p}$.
  3. `BalancedTradeoffSolver` (`BALANCED`): Equal compromise ($\lambda = 0.5$) balancing normalized preference satisfaction against skill fit.
- **Distinct Alternative Generation**: Implemented sequential integer no-good cuts ($\sum_{(s,p) \in A} x_{s,p} \le |S| - 1$), guaranteeing that each subsequent alternative differs for at least one student without fabricating duplicates.
- **Analytical Indicators**: Computation of feasibility checks, efficiency metrics (mean skill fit, soft skill gap), fairness indicators (mean rank, worst rank, unranked count, rank Gini coefficient $G$), and project capacity utilization.
- **Domain Validation & Diagnostics**: Comprehensive pre-flight checks catching capacity deficits, inverted bounds, and contradictory locks, providing structured diagnostic error messages on mathematical infeasibility.

### 1.3 Contract-First In-Process Collaboration Protocol
Our collaboration was governed by an agreed in-process Python contract (`app/contract/contract.py`):
1. **Week 1 Alignment**: Both teams studied and adopted the domain models, SQLite schema and `AssignmentEngineContract` provided in the starter kit.
2. **Parallel Independence**: Team B worked strictly behind the contract interface, while Team A utilized `MockAssignmentEngine` (`app/contract/mocks.py`) to build, test, and style all UI views and controllers without blocking on solver completion.
3. **In-Process Integration**: Rather than introducing network latency, HTTP serialization overhead, or distributed failure modes, the two components integrate directly in Python: Team A instantiates `solvers.AssignmentEngine()` and exchanges strongly typed DTOs (`RunRequest`, `ValidationOutcome`, `RunComparison`, `AssignmentRun`).

---

## 2. Software Development Lifecycle (SDLC) & Team Process

### 2.1 SDLC Milestone Execution
Our development followed an iterative, test-driven 3-week milestone roadmap:
- **Week 1 (Architectural Design & Contract)**: Studied the starter kit and contract. Team A implemented skill and project CRUD; Team B formulated the ILP model and verified input validation.
- **Week 2 (Component Development & Mocks)**: Team B implemented the three assignment strategies and evaluation metrics. Team A built the web interface, role-based access control, and solution comparison tools using the mock engine.
- **Week 3 (Full Integration, Strategy 3 & Verification)**: Connected Component A to the production `AssignmentEngine`. Integrated Strategy 3 (`BALANCED`), replaced earlier ad-hoc test suites with the professor's 50-test reference suite, resolved edge cases, and completed the project documentation.

### 2.2 Git Collaboration & Branch Management

The six team members worked in two GitHub repositories, both forked from the course starter repository:
- **Team A** worked in the shared repository `MariaRogdaberg/TECH5-collaborative-Project`.
 Features were developed on separate branches (e.g. `skills-crud`, `alexnder_todo`, `week-3`) and integrated into `main` mostly through pull requests. Not all pull requests were reviewed by a second team member before merging, as we had sat together and made the code.
- **Team B** developed the Assignment Engine in a separate fork, `JoggiLJ/TECH5-collaborative-Project`. Their work was brought into the shared repository by adding Team B's fork as a Git remote (`teamB`) and merging it on dedicated branches (`merge-team-b-w1` in week 1 and `merge-team-b-w3` in week 3, the latter via PR #8). Team B also merged Team A's work into their own fork, so the two repositories were kept in sync manually from both sides.

Working in two repositories was tidy for the most part. But because both were forks of the starter repository, GitHub suggested the course repository as the target for new pull requests, which led to some pull requests being opened in the wrong place. 

When Team A developed advanced recommendation tests (`tests/test_recommendations.py`, 243 lines), they were verified and merged in PR #7 (commit `3a31b09`). When synchronizing with the professor's reference suite in PR #8, the test configuration was cleanly reconciled without losing commit history.

### 2.3 Disciplined AI Pair Programming Practice
The course allows generative AI in the collaborative project, and both teams used it. We treated AI as an assistant rather than an author: every change was read, understood and tested before it was committed, following the "explain before accept" practice.

**Tools.** Team A used Google Antigravity IDE and Claude.

**What we used AI for:**
- **Understanding the starter code.** AI explained how a request moves from a Flask route through the contract and repositories to the database, and how the repository classes relate to the database exercises from lectures. This was the most valuable use, because it helped us work in code we had not written ourselves.
- **Implementing and debugging features.** AI helped draft the CRUD routes and the comparison and recommendation logic. It also diagnosed errors from the terminal, for example a missing `pulp` package caused by running tests in the conda base environment instead of the project's virtual environment.
- **Git and GitHub.** Setting up the other team's fork as a remote, merging across the two repositories, resolving pull requests that targeted the course repository by mistake, and checking what each merge would change before running it.
- **Review and testing.** Checking the analytical note against the code revealed several mismatches, such as recommendation rules that described a different selection logic than `recommendations.py`. An AI-assisted run through the application with invalid inputs found a crash on non-numeric input for the number of alternatives, which we then fixed.

**Limitations we experienced.** AI suggestions were not always correct or complete. For example, when we asked AI to write a test that checked whether the application handled an empty skills list correctly, it produced a test that failed on that same input because it did not properly handle the data. We therefore verified every change by running `pytest` and testing the workflow in the browser.

**Reflection.** AI made us faster at understanding unfamiliar code and at diagnosing problems, but it did not replace understanding.

---

## 3. Key Technical Challenges & Reflections

### 3.1 Multi-Objective Optimization Trade-Offs
A central insight from building this system is the inherent tension between individual student happiness and project quality. Optimizing exclusively for preferences produces teams that lack essential technical skills, while optimizing purely for skills forces students into low-ranked or unranked projects. Implementing Strategy 3 (`BALANCED` with $\lambda = 0.5$) and calculating the Gini coefficient provided an objective basis for coordinators to evaluate compromise solutions.

### 3.2 Distinct Alternatives via No-Good Integer Cuts
Decision-makers require genuinely distinct alternatives. Fabricating duplicate runs or slight permutations is unacceptable. By formulating integer no-good cuts that exclude previously discovered assignment vectors, the engine guarantees that every subsequent solution differs for at least one student, or accurately reports that no further feasible allocations exist.

### 3.3 Interactive Pinning & Real-Time Re-Solving
Real-world academic allocation requires coordinator discretion (e.g. funding requirements, special pedagogical needs). Allowing coordinators to pin a student to a project directly within the solution view and immediately trigger a re-solve—with the guarantee that the pin is preserved and peer displacement is quantified—proved to be the most compelling feature of the decision support system.

---

### 3.4 Lessons Learned
- **Integrate the real engine early.** The mock engine always succeeded, so failure cases only appeared at integration. Mock fallbacks even hid real failures until we removed them.
- **Review every pull request.** Sitting together while coding is not a substitute for a second person checking the final change.
- **Keep documentation in step with the code.** Reviewing the analytical note against the code revealed several mismatches that had built up as the engine changed.

---

## 4. Self-Assessment against the 100-Point Grading Rubric

| Category | Weight | Self-Score | Evidence & Justification |
|---|:---:|:---:|---|
| **1. Business requirements and analytical formulation** | 10% | **9 / 10** | The analytical note (`docs/analytical_note_template.md`) gives a complete ILP formulation with sets, parameters, five hard constraints, three objectives, evaluation indicators, recommendation rules, assumptions and limitations, and it was reviewed against the code. Weakness: λ is fixed at 0.5 and cannot be changed by the user. |
| **2. Object-oriented architecture and Python implementation** | 15% | **12 / 15** | The engine extends the starter kit's Strategy Pattern with three concrete strategies, and Team A's features use the Repository Pattern and the abstract contract consistently. Weakness: much of the architecture was provided by the starter kit, and `routes.py` has grown to about 1,100 lines and would benefit from being split into separate modules. |
| **3. Relational database design and SQL integration** | 15% | **11 / 15** | All CRUD features persist through the repository classes with parameterized SQL, and the app works on the normalized schema with foreign keys and CHECK constraints. Weakness: the schema and the analytical SQL queries in `docs/database_design.md` came from the starter kit and were not extended by us. Deleting a skill that projects require silently removes those requirements (`ON DELETE CASCADE`) without warning the user. |
| **4. Assignment Engine functionality and analytical quality** | 20% | **18 / 20** | PuLP/CBC engine with three strategies (`PREFERENCE`, `SKILL_FIT`, `BALANCED`), enforcement of capacities, mandatory skills and fixed locks, distinct alternatives via no-good cuts, feasibility, efficiency and fairness indicators including a Gini coefficient, and structured infeasibility diagnostics. Weaknesses: later strategies in a run are constrained by earlier solutions, the ε term is not a strict tie-breaker in Strategy 2, and the comparison function treats unranked students differently from the indicator calculation. |
| **5. Visualization, recommendations, and interaction** | 15% | **13 / 15** | Working CRUD for projects and skills, side-by-side comparison with deltas and changed students, lock-and-rerun from the solution view, and recommendations that separate evidence from advice across three policy priorities. Weakness: the trade-off weight λ and the number of strategies cannot be explored interactively. |
| **6. Component contract and end-to-end integration** | 10% | **9 / 10** | Team A developed against `MockAssignmentEngine` and switched to the real engine through the contract without changing the interface. We removed the mock fallbacks so that real failures are reported instead of hidden. Weakness: integration happened late and across two repositories. |
| **7. Testing, error handling, and robustness** | 10% | **8 / 10** | All 50 tests in the professor's reference suite pass (35 passed, 1 skipped, 15 xpassed), including all edge cases. Manual testing with invalid input found and fixed a crash on non-numeric input for the number of alternatives. Weakness: Team A's own recommendation tests were retired when the reference suite was restored, so recommendation logic is now covered only indirectly. |
| **8. Git collaboration, SDLC practice, and team process** | 5% | **3 / 5** | Feature branches and pull requests were used throughout Team A's work. Weaknesses: the teams worked in two separate forks that had to be merged manually, some work was merged without a pull request, and not all pull requests were reviewed by a second person. |
| **Total** | **100%** | **83 / 100** | |