# Savings Goals Feature Flowchart
## 1. Purpose
- Creating a goal
- Updating progress on a goal
- Retrieving goals (all for a user or a single goal)
- Calculating goal status (On Track / At Risk / Overdue / Achieved)
- Sending notifications based on proximity to target date

## 2. Notation and structure (how the diagram is organized)
- A top section representing the **data store** (`savings_goals`) and basic start/end.
  - `create_savings_goal(user_id, savings_goal_name, target_amount, target_date)`
  - `update_savings_progress(user_id, goal_id, amount_to_add)`
  - `retrieve_savings_goals(user_id, goal_id=None)`
  - `calculate_savings_goal_status(goal)`
  - `send_savings_goal_notification(user_id, goal_id)`

This approach keeps each workflow readable and lets you trace logic *within a function* without mixing it with other functions’ internal steps.

## 3. Data model used in the flowchart
- `goal[0]` = `goal_id`
- `goal[1]` = `user_id`
- `goal[2]` = `name`
- `goal[3]` = `target_amount`
- `goal[4]` = `target_date`
- `goal[5]` = `amount_saved`
- `goal[6]` = `creation_date`

The “data store” is modeled as:
- `savings_goals = []` (a list of goal records)

## 4. How each sub-flow was crafted

### 4.1 `create_savings_goal(...)`
**Intent:** Create a new goal record and persist it.

**Key steps represented:**
1. Start
2. Generate a unique `goal_id`
3. Build `new_goal` with default `amount_saved = 0` and `creation_date = today`
4. Append to `savings_goals`
5. Return the created goal
6. End

### 4.2 `update_savings_progress(...)`
**Intent:** Add money to a goal’s saved amount.

**Key steps represented:**
1. Start
2. Loop through `savings_goals`
3. Decision: match on both `goal_id` and `user_id`
4. If match → update `goal[5] += amount_to_add` and return updated goal
5. If loop completes without match → return `None`
6. End

### 4.3 `retrieve_savings_goals(user_id, goal_id=None)`
**Intent:** Fetch either all goals for a user or a single goal by id.

**Key steps represented:**
1. Start
2. Initialize `user_goals = []`
3. Loop over `savings_goals` and filter by `goal[1] == user_id`
4. After collecting, decision: `goal_id provided?`
   - If **No** → return `user_goals`
   - If **Yes** → loop over `user_goals` and look for `goal[0] == goal_id`
     - If found → return goal
     - If not found → return `None`
5. End

### 4.4 `calculate_savings_goal_status(goal)`
**Intent:** Compute a status label for a goal.

**Key steps represented:**
1. Start; set `today`
2. Compute `days_remaining` and `amount_remaining`
3. Decision: `amount_remaining <= 0?` → **Achieved**
4. Else decision: `days_remaining <= 0?` → **Overdue**
5. Else compute a `required_daily_saving = amount_remaining / days_remaining`
6. Compute `days_since_creation`
   - If `days_since_creation > 0` → compute `current_daily_saving_rate = goal[5] / days_since_creation`
   - Else set `current_daily_saving_rate = 0`
7. Decision: `current_daily_saving_rate >= required_daily_saving?`
   - Yes → **On Track**
   - No → **At Risk**
8. End

### 4.5 `send_savings_goal_notification(user_id, goal_id)`
**Intent:** Notify users when a goal is close to due or overdue.

**Key steps represented:**
1. Start
2. Call `retrieve_savings_goals(user_id, goal_id)`
3. Decision: goal exists?
   - No → print “Goal not found” → End
4. If yes, compute `days_to_target` and `amount_needed`
5. Decision: `0 < days_to_target <= 7 AND amount_needed > 0?`
   - Yes → build reminder → `send_notification(user_id, reminder)`
   - No → decision: `days_to_target <= 0 AND amount_needed > 0?`
     - Yes → build alert → `send_notification(user_id, alert)`
     - No → End

## 5. Cross-function linking (“calls”)
The flowchart includes explicit call relationships:
- `send_savings_goal_notification(...)` **calls** `retrieve_savings_goals(...)`

## 6. Known assumptions / simplifications
- Data store is depicted as an in-memory list rather than a real database table.
- Record fields are positional (`goal[5]`) rather than named (`goal.amount_saved`).
- Notification delivery is abstracted as `send_notification(...)`.


