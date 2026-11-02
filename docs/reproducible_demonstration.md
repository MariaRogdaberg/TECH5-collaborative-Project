# TECH5 Assignment Support System: Reproducible Demonstration Walkthrough

## 1. Demonstration Overview

This guide provides instructors, evaluators, and students with an exact, reproducible walkthrough of the **Student-to-Project Assignment Decision Support System**. It exercises all core functional capabilities across the four benchmark cohorts pre-configured in the platform.

### Quick Start
1. Create and activate the virtual environment, and install dependencies:
   ```bash
   python -m venv .venv
   source .venv/bin/activate        # macOS/Linux
   # .\.venv\Scripts\activate       # Windows
   pip install -r requirements.txt
   ```
2. Start the application: `python run.py`
3. Open **`http://127.0.0.1:5000`** in your browser.
---

## 2. Scenario 1: Standard Cohort (Trade-Off Benchmark)

### 2.1 Scenario Objective
Demonstrates distinct alternatives, one per strategy, through the generation of distinct solutions via no-good integer cuts, and comparative trade-off analysis between student preference satisfaction and project skill fit.

### 2.2 Cohort Composition
- **Students**: 10 students (`S01` to `S10`) with complementary competencies across Python, SQL, Machine Learning, Web Dev, and FinTech.
- **Projects**: 4 projects (`P1`: Sustainable FinTech Risk Engine, `P2`: Cloud Data Pipeline & Lakehouse, `P3`: Healthcare Predictive ML Assistant, `P4`: Smart City Mobility Portal), each with capacity bound $[1, 3]$ (Total Capacity = 12 $\ge 10$ students).
- **Constraints**: No fixed pre-allocations initially.

### 2.3 Step-by-Step Execution (Web Interface)
1. In the top navigation, click **Dashboard** or **Scenarios**.
2. Click the green button **"Load Standard Benchmark"** under **Standard Trade-Off Benchmark**.
3. A success notification will confirm the cohort is loaded into SQLite (10 students, 4 projects (capacity 3 each)).
4. In the **Optimization Run Trigger Panel**:
   - Strategy: `Student Preference Maximization (Minimize Rank Regret)`
   - Distinct Alternatives: `3 Distinct Solutions`
   - Click **"Launch Optimization Run"**.
5. You are redirected to `/runs/{run_id}`:
   - Notice that **3 distinct solutions** have been computed and persisted.
   - **Alternative 1 (Preference-Oriented)**:
     - Mean Skill Fit: ~`98.3%`
     - Mean Preference Rank: ~`1.00` (All 10 students get 1st choice)
     - Worst Rank: `1`
     - Gini Inequality: `0.000`
   - **Alternative 2 (Skill-Fit Oriented)**:
     - Mean Skill Fit increases to ~`100.0%`
     - Mean Preference Rank adjusts to ~`1.10` (9 students get 1st choice, 1 student gets 2nd choice)
     - Worst Rank: `2`
     - Gini Inequality: `0.082`
   - **Alternative 3 (Balanced Trade-Off)**:
     - Mean Skill Fit: ~`98.3%`
     - Mean Preference Rank: ~`1.10` (9 students get 1st choice, 1 student gets 2nd choice)
     - Worst Rank: `2`
     - Gini Inequality: `0.082`

### 2.4 Side-by-Side Solution Comparison
1. In the top navigation, click **Compare**.
2. Select:
   - Solution A: `Alternative 1 (PREFERENCE)`
   - Solution B: `Alternative 2 (SKILL_FIT)`
   - Click **"Compare Allocations →"**.
3. The comparison dashboard displays:
   - **Delta Metrics**: Visual indicator badges displaying $\Delta$ Mean Skill Fit (+1.7%), $\Delta$ Mean Rank (+0.10), $\Delta$ Worst Rank (+1.0) and $\Delta$ Gini (+0.082)
   - **Re-Allocation Table**: Highlights that only S06 moves, from P3 (first choice) to P2 (2nd choice), along with their individual rank change ($\Delta Rank$).

### 2.5 Inspecting Decision-Support Recommendations
1. On the run page, click **"Decision-Support Recommendations Available"**.
2. The page shows one recommendation per policy priority, each separating calculated evidence from policy advice and listing the trade-offs:
   - **Student Preference Maximization** recommends Alternative #1 (everyone gets their first choice).
   - **Project Skill Quality** recommends Alternative #2 (perfect skill fit, at the cost of one student's first choice).
   - **Balanced & Egalitarian Allocation** also recommends Alternative #1, since it has the lowest Gini coefficient (0.0).
3. Note that the balanced recommendation is not the BALANCED solution (#3). Recommendations compare all alternatives on the stated criteria, regardless of which strategy produced them.

### 2.6 Headless In-Process Contract Execution (Python)
The Assignment Engine can be invoked directly in Python without starting the web server, demonstrating contract independence:
```python
from app.contract import contract
from app.engine import solvers
from app.persistence import database
from app.samples import demo_data

# Create the database tables if they do not exist yet
database.DatabaseManager().initialize_schema()

# Initialize the engine
engine = solvers.AssignmentEngine()

# Prepare structured run request from the standard demo cohort
data = demo_data.get_standard_scenario()
request = contract.RunRequest(
    students=data["students"],
    projects=data["projects"],
    skills=data["skills"],
    fixed_assignments=data["fixed_assignments"],
    strategy_key="PREFERENCE",
    max_alternatives=3,
)

# Execute optimization via the internal contract
outcome = engine.execute_run(request)
print(f"Status: {outcome.status}")
print(f"Solutions generated: {len(outcome.solutions)}")
for sol in outcome.solutions:
    print(f"Alt {sol.alternative_number} ({sol.strategy_name}): "
          f"Mean Rank={sol.indicators['fairness']['mean_rank']}, "
          f"Mean Skill Fit={sol.indicators['efficiency']['mean_skill_fit']:.2f}")
```
---

## 3. Scenario 2: Fixed Assignment & Cost of a Lock

### 3.1 Scenario Objective
Demonstrates administrative pre-allocation locks ($x_{s^*, p^*} = 1$), strict constraint preservation, and the resulting cost in the fairness indicators.

### 3.2 Cohort Composition
- Base standard cohort with student **`S02` (Bob Smith)** locked to project **`P2` (Cloud Data Pipeline)** under an approved research grant, even though Bob's #1 preference is `P1`.

### 3.3 Step-by-Step Execution
1. Navigate to **Scenarios** and click **"Load Fixed Assignment Scenario →"**.
2. In the top navigation, click **Fixed Locks**.
   - Notice `S02` $\to$ `P2` is listed with status `APPROVED` and the justification "Pre-arranged research grant fellowship on cloud data pipelines."
3. Return to **Dashboard** and click **"Launch Optimization Run →"**.
4. In Alternative 1, `S02` is assigned to `P2` (his 3rd choice) while all other students still get their 1st choice. The cost of the lock shows in the indicators: mean rank rises from `1.00` to `1.20`, and worst rank from `1` to `3`.
5. Click **"🔒 Lock & Rerun"** next to any other student in the solution view to add a second lock and re-run. Both locks are preserved.

---

## 4. Scenario 3: Capacity Deficit (Mathematical Infeasibility)

### 4.1 Scenario Objective
Validates that the system rigorously detects structural infeasibility when total project capacity is strictly less than cohort size ($\sum u_p < |S|$), halting before solver failure and providing clear diagnostic guidance.

### 4.2 Cohort Composition
- **Students**: 10 students.
- **Projects**: Capacities reduced to 2, 2, 1, 1 (Total Capacity = 6 seats for 10 students).

### 4.3 Step-by-Step Execution
1. Navigate to **Scenarios** and click **"Load Infeasible Capacity Scenario →"**.
2. On the Dashboard KPI cards:
   - Total Max Capacity: `6`, marked **"⚠ Capacity Deficit"**, for a cohort of 10 students.
3. Click **"Launch Optimization Run →"**.
4. The system detects the infeasibility before running the solver:
   - Run Status: `INFEASIBLE`
   - Run Diagnostics:
     ```
     Input validation proved the problem infeasible, so the solver was not run.
     Insufficient overall capacity: total cohort size (10) exceeds aggregate maximum project capacity (6).
     ```
   - No solutions are created. Because every student must receive exactly one project, the coordinator must increase project capacities or reduce the cohort before a valid allocation exists.

---

## 5. Scenario 4: Contradictory Fixed Constraints

### 5.1 Scenario Objective
Verifies that input validation rejects contradictory fixed assignments, here when an administrator locks more students to a project than its maximum capacity ($|F_p| > u_p$).

### 5.2 Cohort Composition
- Project `P1` has a maximum capacity of 3 students.
- 4 students (`S01`, `S02`, `S05`, `S08`) are locked to `P1`.

### 5.3 Step-by-Step Execution
1. Navigate to **Scenarios** and click **"Load Contradictory Scenario →"**.
2. Note that the Dashboard shows **"✓ Sufficient Capacity"** (12 places for 10 students). The problem is not total capacity, but the locks on a single project.
3. Click **"Launch Optimization Run →"**.
4. Input validation catches the contradiction before the solver runs:
   - Run Status: `INFEASIBLE`
   - Run Diagnostics:
     ```
     Input validation proved the problem infeasible, so the solver was not run.
     Fixed assignments (4) for project 'P1' exceed its maximum capacity (3).
     ```
   - No solutions are created. The coordinator must release one of the locks on the **Fixed Locks** page before running again.