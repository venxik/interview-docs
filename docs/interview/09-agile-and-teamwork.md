# Agile & Teamwork

This guide covers Agile methodologies, Scrum practices, Git workflows, and teamwork strategies.

---

## Topics Covered

- Agile principles and practices
- Scrum framework and ceremonies
- Git workflows and branching strategies
- Team collaboration
- Task prioritization
- Definition of Done
- Conflict resolution

---

## **Category 18: Agile & Team Dynamics**

These questions test your understanding of Agile methodologies and how you work in team environments.

---

**72. What is Agile? How does it differ from Waterfall?**

- **Answer:** "Agile is an iterative software development methodology that emphasizes flexibility, collaboration, and customer feedback.

**Waterfall vs Agile:**

| Aspect | Waterfall | Agile |
|--------|-----------|-------|
| **Approach** | Sequential, linear | Iterative, incremental |
| **Planning** | Upfront, detailed | Ongoing, adaptive |
| **Requirements** | Fixed at start | Evolve over time |
| **Testing** | End of development | Continuous |
| **Customer involvement** | Start and end | Throughout process |
| **Delivery** | One big release | Frequent small releases |
| **Flexibility** | Rigid, hard to change | Flexible, embraces change |
| **Risk** | High (discover issues late) | Lower (discover issues early) |

**Waterfall Process:**

```
Requirements → Design → Implementation → Testing → Deployment → Maintenance
     ↓            ↓            ↓            ↓          ↓           ↓
  Months       Months       Months       Months     Weeks       Ongoing
```

**Agile Process (Scrum):**

```
Sprint 1 (2 weeks): Plan → Design → Code → Test → Review → Deploy
Sprint 2 (2 weeks): Plan → Design → Code → Test → Review → Deploy
Sprint 3 (2 weeks): Plan → Design → Code → Test → Review → Deploy
... (continuous cycles)
```

**Key Agile Principles:**

1. **Individuals and interactions** over processes and tools
2. **Working software** over comprehensive documentation
3. **Customer collaboration** over contract negotiation
4. **Responding to change** over following a plan

**Real-world example:**

**Waterfall approach:**
- Spend 3 months gathering all requirements
- Spend 2 months designing entire system
- Spend 6 months building everything
- Spend 2 months testing
- Deploy everything at once
- **Problem:** Customer sees product after 13 months, requirements may have changed

**Agile approach:**
- Week 1-2: Build login feature, test, deploy
- Week 3-4: Build user dashboard, test, deploy
- Week 5-6: Build data upload, test, deploy
- **Benefit:** Customer sees working features every 2 weeks, can provide feedback early

**In this project:**

If this project were developed using Agile:
- **Sprint 1:** Basic models and database setup
- **Sprint 2:** Class management API (GET, POST)
- **Sprint 3:** Student management API
- **Sprint 4:** CSV upload feature
- **Sprint 5:** Workload report feature
- Each sprint delivers working, testable functionality"

---

**73. Explain Scrum. What are sprints, user stories, and daily standups?**

- **Answer:** "Scrum is the most popular Agile framework for managing software development projects.

**Key Scrum Concepts:**

**1. Sprints**

- Fixed-length iteration (usually 1-4 weeks, commonly 2 weeks)
- Each sprint delivers a potentially shippable product increment
- Timeboxed: Sprint length doesn't change

**Sprint Structure:**

```
Sprint (2 weeks):
Day 1:     Sprint Planning (4 hours)
Day 2-9:   Daily Standup (15 min) + Development Work
Day 10:    Sprint Review/Demo (2 hours)
           Sprint Retrospective (1.5 hours)
           (Optional: Next Sprint Planning)
```

**2. User Stories**

A user story describes a feature from the end-user's perspective:

**Format:** "As a [type of user], I want [goal] so that [benefit]"

**Examples from this project:**

```
✅ As a school administrator, I want to upload student data via CSV
   so that I can quickly onboard many students at once.

   Acceptance Criteria:
   - Can upload .csv file
   - System validates email format
   - Shows success/error messages
   - Updates database

✅ As a school administrator, I want to view teacher workload reports
   so that I can balance teaching assignments fairly.

   Acceptance Criteria:
   - Shows each teacher's name
   - Lists subjects taught
   - Shows number of classes per subject
   - Data is accurate

✅ As a teacher, I want to see all students in my class
   so that I can prepare my lesson plans.

   Acceptance Criteria:
   - Shows student names
   - Shows student emails
   - Can filter by class
   - Can export list
```

**Story Points (Estimation):**

- Relative measure of complexity/effort
- Common scale: 1, 2, 3, 5, 8, 13, 21 (Fibonacci)
- Example:
  - "Add new field to form" → 2 points (simple)
  - "Implement CSV upload" → 8 points (complex)
  - "Build entire authentication system" → 21 points (very complex, should be broken down)

**3. Daily Standup (Daily Scrum)**

- 15-minute meeting, same time daily
- Team stands (keeps it short!)
- Each person answers:
  1. **What did I do yesterday?**
  2. **What will I do today?**
  3. **Are there any blockers?**

**Example Standup:**

```
Developer 1:
- Yesterday: Finished CSV parser, wrote tests
- Today: Integrate CSV parser with DataUploadService
- Blockers: None

Developer 2:
- Yesterday: Worked on workload report API
- Today: Fix bug with duplicate class counting, deploy to staging
- Blockers: Waiting for staging database credentials

Developer 3:
- Yesterday: Reviewed pull requests, deployed hotfix
- Today: Start work on student enrollment feature
- Blockers: Need clarification on business logic for transfers
```

**4. Scrum Roles:**

**Product Owner:**
- Defines features and prioritizes backlog
- Represents customer/business needs
- Makes final decisions on what gets built

**Scrum Master:**
- Facilitates Scrum ceremonies
- Removes blockers
- Protects team from interruptions
- Coaches team on Agile practices

**Development Team:**
- Cross-functional (developers, testers, designers)
- Self-organizing
- Typically 5-9 people
- Commits to sprint goals

**5. Scrum Ceremonies:**

**Sprint Planning:**
- Team selects user stories from backlog
- Breaks stories into tasks
- Commits to sprint goal

**Daily Standup:**
- Synchronize team
- Identify blockers

**Sprint Review/Demo:**
- Show completed work to stakeholders
- Gather feedback

**Sprint Retrospective:**
- What went well?
- What could be improved?
- Action items for next sprint

**In this project:**

A Scrum sprint might look like:

**Sprint Goal:** "Implement and deploy CSV data upload feature"

**User Stories (committed):**
1. Parse CSV files (5 points)
2. Validate data format (3 points)
3. Insert data to database (5 points)
4. Show error messages (2 points)
5. Write tests (3 points)
**Total:** 18 points

**Daily progress:**
- Day 1: Sprint planning, start CSV parsing
- Day 2-3: Complete CSV parsing, start validation
- Day 4-5: Complete validation, start database insertion
- Day 6-8: Complete database insertion, write tests
- Day 9: Bug fixes, integration testing
- Day 10: Sprint review (demo to stakeholders), retrospective"

---

**74. What is a retrospective? Why is it important?**

- **Answer:** "A Sprint Retrospective is a meeting held at the end of each sprint where the team reflects on how they worked and identifies improvements.

**Purpose:**

- Continuous improvement of team processes
- Build team cohesion
- Address issues before they become bigger problems

**Structure (1-1.5 hours):**

**1. Set the Stage (5 min)**
- Review retrospective goals
- Establish safe environment (no blame)

**2. Gather Data (15 min)**
- What happened this sprint?
- Review metrics (velocity, bugs, deployment frequency)

**3. Generate Insights (20 min)**
- Why did things happen?
- What patterns do we see?

**4. Decide Actions (20 min)**
- What will we do differently?
- Commit to 1-3 actionable improvements

**5. Close (5 min)**
- Summarize action items
- Assign owners

**Common Retrospective Formats:**

**Format 1: Start, Stop, Continue**

```
START doing:
- Code reviews within 24 hours
- Writing tests before pushing

STOP doing:
- Working on weekends
- Skipping standup meetings

CONTINUE doing:
- Pair programming on complex features
- Documenting architectural decisions
```

**Format 2: What Went Well, What Didn't, Actions**

```
What Went Well:
✅ CSV upload feature deployed on time
✅ Zero production bugs this sprint
✅ Good collaboration on workload report

What Didn't Go Well:
❌ Staging environment was down for 2 days
❌ Unclear requirements for student enrollment
❌ Too many meetings interrupted deep work

Actions:
1. [Owner: DevOps] Set up monitoring for staging
2. [Owner: Product Owner] Schedule requirements workshop
3. [Owner: Scrum Master] Block mornings for focus time
```

**Format 3: 4 Ls (Liked, Learned, Lacked, Longed For)**

```
LIKED:
- New testing framework is fast
- Team lunch on Friday

LEARNED:
- How to optimize database queries
- New TypeScript features

LACKED:
- Clear definition of "done"
- Automated deployment pipeline

LONGED FOR:
- Better dev environment setup
- More time for tech debt
```

**Example Retrospective Conversation:**

**Scrum Master:** "What went well this sprint?"

**Dev 1:** "I liked how we pair-programmed on the CSV upload. We caught bugs early and both learned from each other."

**Scrum Master:** "Great! What didn't go well?"

**Dev 2:** "The staging database went down, which blocked our testing for 2 days."

**Dev 3:** "Also, we had unclear requirements for how to handle duplicate emails. We coded it one way, then had to refactor."

**Scrum Master:** "What can we do to prevent these issues?"

**Team discusses...**

**Actions:**
1. ✅ [DevOps] Set up automated health checks for staging (by next sprint)
2. ✅ [Product Owner] Add acceptance criteria checklist to story template (by tomorrow)
3. ✅ [Team] Schedule 30-min requirements clarification at start of sprint planning (ongoing)

**Why Retrospectives Matter:**

1. **Continuous Improvement:** Small improvements compound over time
2. **Team Ownership:** Team decides how to improve (not imposed from above)
3. **Psychological Safety:** Creates space to discuss problems openly
4. **Prevents Burnout:** Addresses team morale and workload issues
5. **Knowledge Sharing:** Team learns from each other's experiences

**Common Mistakes:**

❌ No action items (just complaining)
❌ Same issues every sprint (not following through)
❌ Blaming individuals (focus on processes)
❌ Too many action items (focus on 1-3 max)
❌ Skipping retrospectives when busy (that's when you need them most!)

**In this project:**

A retrospective after completing the CSV upload feature might surface:

- ✅ **Liked:** Good test coverage prevented bugs
- ❌ **Lacked:** Documentation on CSV format for users
- 🎯 **Action:** Create API documentation and CSV template examples"

---

**75. What is your experience with Git and version control workflows?**

- **Answer:** "I have experience with Git and follow best practices for version control in team environments.

**Git Workflow (Feature Branch Workflow):**

```
main (production-ready code)
  ↓
develop (integration branch)
  ↓
feature/csv-upload (your feature)
```

**Typical Workflow:**

```bash
# 1. Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/csv-upload

# 2. Make changes
git add src/services/DataUploadService.ts
git commit -m "feat: implement CSV parser for student data"

# 3. Keep feature branch updated
git fetch origin
git rebase origin/develop  # or merge, depending on team preference

# 4. Push feature branch
git push origin feature/csv-upload

# 5. Create Pull Request on GitHub
# 6. Address code review feedback
# 7. Merge to develop after approval

# 8. Delete feature branch
git branch -d feature/csv-upload
git push origin --delete feature/csv-upload
```

**Commit Message Convention (Conventional Commits):**

```bash
# Format: <type>(<scope>): <description>

feat(upload): add CSV parser for student data
fix(api): handle null values in workload report
refactor(repo): simplify query logic in ClassRepository
test(service): add tests for DataUploadService
docs(readme): update setup instructions
chore(deps): update Sequelize to v6.35.0
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `test`: Adding or updating tests
- `docs`: Documentation changes
- `chore`: Maintenance tasks (dependencies, config)
- `perf`: Performance improvement
- `style`: Code style changes (formatting, no logic change)

**Branching Strategy:**

**1. Main branches:**
- `main`: Production code (always deployable)
- `develop`: Integration branch (latest features)

**2. Feature branches:**
- `feature/feature-name`: New features
- `fix/bug-description`: Bug fixes
- `refactor/description`: Code refactoring
- `hotfix/critical-bug`: Urgent production fixes

**Example in this project:**

```
main (v1.0.0)
  └─ hotfix/fix-email-validation (emergency fix)

develop (v1.1.0-dev)
  ├─ feature/csv-upload (Developer 1)
  ├─ feature/workload-report (Developer 2)
  └─ fix/class-count-bug (Developer 3)
```

**Pull Request Best Practices:**

**Good PR Description:**

```markdown
## Summary
Implements CSV data upload feature for teachers, students, and classes.

## Changes
- Add CSV parser utility (`src/utils/csvParser.ts`)
- Implement DataUploadService with validation
- Add upload endpoint to UploadDataController
- Handle error cases (invalid format, duplicate emails)

## Testing
- Unit tests for CSV parser
- Integration tests for upload endpoint
- Tested with sample data (1000 students, 50 classes)

## Screenshots
[Screenshot of success/error messages]

## Related Issues
Closes #45
Related to #23

## Checklist
- [x] Tests added/updated
- [x] Documentation updated
- [x] Code follows style guide
- [x] No console.log statements
- [x] Tested locally
```

**Code Review (as reviewer):**

When reviewing PRs, I check:

1. **Functionality:** Does it work as intended?
2. **Tests:** Are there sufficient tests?
3. **Code Quality:** Is it readable and maintainable?
4. **Performance:** Any obvious bottlenecks?
5. **Security:** Any vulnerabilities?
6. **Documentation:** Are comments/docs updated?

**Example review comments:**

```typescript
// ✅ Good feedback
"Consider using a Set here for O(1) lookup instead of Array.find() which is O(n)"

"This validation logic could be extracted to a separate function for reusability"

"Great job adding error handling! Could we also log this error for debugging?"

// ❌ Bad feedback
"This is wrong" (not helpful, no explanation)
"Why didn't you do it this way?" (assumes without context)
"Just fix it" (not constructive)
```

**Merge Strategies:**

**1. Merge Commit (preserves full history):**
```bash
git merge feature/csv-upload
# Creates merge commit
```

**2. Squash and Merge (cleaner history):**
```bash
git merge --squash feature/csv-upload
# Combines all commits into one
```

**3. Rebase (linear history):**
```bash
git rebase develop
# Replays commits on top of develop
```

**Handling Merge Conflicts:**

```bash
# Conflict occurs during merge
git status  # See conflicting files

# Edit files to resolve conflicts
# Look for <<<<<<< HEAD markers

# After resolving:
git add resolved-file.ts
git commit -m "chore: resolve merge conflicts"
```

**In this project:**

Following Git best practices would mean:

1. Each feature (CSV upload, workload report) gets its own branch
2. Frequent commits with clear messages
3. Pull requests reviewed by at least one team member
4. CI/CD runs tests automatically on each PR
5. Squash commits before merging to keep `main` clean"

---

**76. How do you handle disagreements with team members about technical decisions?**

- **Answer:** "I approach technical disagreements professionally and constructively:

**Step 1: Listen and Understand**

- Ask clarifying questions to understand their perspective
- Assume positive intent
- Focus on the technical problem, not personalities

**Example:**

❌ "Your solution is wrong"
✅ "I see your approach. Can you walk me through why you chose that method?"

**Step 2: Present Your Case with Evidence**

- Explain your reasoning with facts:
  - Performance benchmarks
  - Code examples
  - Industry best practices
  - Documentation

**Example Scenario:**

**Disagreement:** Should we use ORM (Sequelize) or raw SQL?

**My approach:**

```
"I understand you prefer raw SQL for performance. Here's my thinking:

PROS of Sequelize (ORM):
✅ Database abstraction (easier to switch databases)
✅ Protection against SQL injection
✅ Faster development for standard queries
✅ Better type safety with TypeScript
✅ Built-in validation

CONS:
❌ Performance overhead (~10-15% slower)
❌ Can't easily write complex queries
❌ Learning curve

PROS of Raw SQL:
✅ Maximum performance
✅ Full control over queries
✅ No abstraction layer

CONS:
❌ Tied to specific database
❌ Must handle SQL injection manually
❌ More boilerplate code

For this project, I suggest Sequelize because:
1. We're not at scale where 10-15% matters (< 10k users)
2. Development speed is priority (startup environment)
3. Team is more familiar with ORMs

However, for the workload report query which is complex, we could use raw SQL within Sequelize:

```typescript
await sequelize.query(`
  SELECT t.name, COUNT(DISTINCT c.id) as class_count
  FROM teachers t
  JOIN teacher_class_subject tcs ON t.id = tcs.teacher_id
  JOIN classes c ON tcs.class_id = c.id
  GROUP BY t.id
`, { type: QueryTypes.SELECT });
```

This gives us the best of both worlds. What do you think?"
```

**Step 3: Seek Common Ground**

- Find areas of agreement
- Propose compromises
- Focus on shared goals (project success)

**Step 4: Defer to Data / Experiment**

If still disagreeing:

```
"How about we prototype both approaches?
- I'll implement ORM version
- You implement raw SQL version
- We benchmark performance, LOC, and development time
- Then decide based on data"
```

**Step 5: Accept Team Decision**

- Once a decision is made, commit fully
- Don't say "I told you so" if it fails
- If you're wrong, acknowledge and learn

**Step 6: Escalate If Necessary**

If disagreement is blocking progress:

- Involve tech lead or architect
- Present both options objectively
- Accept their decision

**Real Example from a Team:**

**Scenario:** Teammate wants to refactor entire codebase to use functional programming

**My response:**

```
"I appreciate your enthusiasm for functional programming. I agree it has benefits.

However, I have concerns about refactoring everything right now:

1. Risk: Large refactors introduce bugs
2. Time: We have feature deadlines coming up
3. Team: Not everyone is familiar with FP patterns
4. Maintenance: New team members will have learning curve

Alternative proposal:
- Use FP patterns for NEW code going forward
- Gradually refactor high-risk areas (not all at once)
- Run a team workshop on FP to get everyone up to speed
- Revisit in 3 months to assess impact

This way we get FP benefits while minimizing risk. Thoughts?"
```

**Key Principles:**

1. **Assume Positive Intent:** They want what's best for the project
2. **Focus on Facts:** Use data, not opinions
3. **Be Humble:** You might be wrong
4. **Be Respectful:** Disagreement ≠ disrespect
5. **Compromise:** Find middle ground
6. **Document:** Record decision and rationale (Architecture Decision Records)

**When to Push Back Hard:**

Some decisions warrant strong disagreement:

- Security vulnerabilities
- Obvious performance issues
- Violation of core principles
- Technical debt that will cripple project

**Example:**

"I strongly disagree with storing passwords in plain text. This is a security risk that could lead to data breaches and legal issues. I recommend we use bcrypt hashing, which is industry standard. This is non-negotiable from a security standpoint."

**In this project:**

If disagreeing about the Layered Architecture choice, I would:

1. Listen to alternative (e.g., microservices)
2. Present trade-offs of each approach
3. Consider project constraints (team size, deadline, scale)
4. Propose a decision matrix
5. Accept team decision and commit fully"

---

**77. How do you prioritize tasks when you have multiple deadlines?**

- **Answer:** "I use a systematic approach to prioritize tasks effectively:

**1. Assess All Tasks**

List everything that needs to be done:

```
Tasks:
1. Fix critical bug in production
2. Complete CSV upload feature
3. Code review for teammate
4. Update API documentation
5. Refactor ClassRepository
6. Attend sprint planning meeting
```

**2. Prioritize Using Eisenhower Matrix**

Categorize by **Urgency** and **Importance**:

```
┌─────────────────────┬─────────────────────┐
│  URGENT + IMPORTANT │ NOT URGENT + IMP.   │
│  (Do First)         │ (Schedule)          │
│                     │                     │
│ • Fix prod bug      │ • CSV upload        │
│ • Sprint planning   │ • Documentation     │
├─────────────────────┼─────────────────────┤
│  URGENT + NOT IMP.  │ NOT URG + NOT IMP.  │
│  (Delegate/Minimize)│ (Eliminate)         │
│                     │                     │
│ • Some meetings     │ • Over-engineering  │
│ • Interruptions     │ • Perfect code      │
└─────────────────────┴─────────────────────┘
```

**3. Consider Impact and Effort**

**High Impact + Low Effort** → Do immediately
**High Impact + High Effort** → Schedule dedicated time
**Low Impact + Low Effort** → Batch together
**Low Impact + High Effort** → Reconsider if necessary

**Example Matrix:**

| Task | Impact | Effort | Priority |
|------|--------|--------|----------|
| Fix production bug | High | Low | 1 (Do now) |
| CSV upload feature | High | High | 2 (Schedule blocks) |
| Code review | Medium | Low | 3 (Do between tasks) |
| Refactor | Low | High | 4 (Defer to next sprint) |
| Documentation | Medium | Low | 5 (End of day) |

**4. Communicate with Stakeholders**

If I can't complete everything:

```
Email to Product Owner:

"Hi [Name],

I have 3 high-priority tasks this week:
1. Production bug fix (blocking users)
2. CSV upload feature (sprint commitment)
3. API documentation (dependency for frontend team)

Given the time constraints, I can deliver #1 and #2 by Friday, but #3 will need to wait until Monday.

If #3 is critical, I can:
- Option A: Defer repository refactoring to next sprint
- Option B: Get help from [teammate] on #2

Please let me know which approach works best."
```

**5. Time Blocking**

Allocate specific time blocks:

```
Monday:
9-10am:   Sprint planning (can't move)
10-12pm:  Fix production bug
1-3pm:    Start CSV upload feature
3-4pm:    Code review
4-5pm:    Respond to questions/emails

Tuesday:
9-12pm:   CSV upload (deep work block)
1-2pm:    Update documentation
2-4pm:    Testing and deployment
4-5pm:    Buffer for unexpected issues
```

**6. Use the 2-Minute Rule**

If a task takes < 2 minutes, do it immediately:

- Quick code review
- Answer a question
- Fix a typo

Don't add these to your task list—just do them.

**7. Handle Interruptions**

```
Teammate: "Can you help me debug this issue?"

❌ Bad: "Sure!" (drops everything)

✅ Good:
"I'm in the middle of fixing a production bug. Can I help you in 30 minutes?"

or

"Is it blocking you completely? If yes, I can help now. If not, let's schedule for 2pm."
```

**Real-World Example:**

**Situation:** Friday afternoon, three urgent issues:

1. Production bug (customers can't log in)
2. Feature due today (CSV upload)
3. Deployment pipeline broken (team blocked)

**My prioritization:**

```
Analysis:
1. Prod bug: Blocks ALL users → Highest priority
2. Pipeline: Blocks entire team → Second priority
3. CSV feature: Only blocks me → Can slip to Monday

Decision:
- 2-3pm: Fix production bug (affects users NOW)
- 3-4pm: Fix deployment pipeline (unblocks team)
- 4-5pm: Communicate feature delay to PM
- Monday: Finish CSV feature

Outcome:
- Users unblocked by 3pm
- Team unblocked by 4pm
- Feature delivered Monday with better quality (not rushed)
```

**In this project:**

If I had to prioritize:

**Sprint backlog:**
1. CSV upload feature (8 pts) - Sprint commitment
2. Workload report API (5 pts) - Sprint commitment
3. Add class filter (3 pts) - Nice to have

**Unexpected:**
- Production bug: Email validation failing

**My prioritization:**

```
Day 1-2:  Fix production bug (unblocks users)
Day 3-6:  CSV upload (committed to sprint)
Day 7-9:  Workload report (committed to sprint)
Day 10:   Sprint review/retro

Outcome: Defer "Add class filter" to next sprint (communicate early!)
```

**Key Principles:**

1. ✅ **Impact over Effort:** Prioritize high-impact tasks
2. ✅ **Communicate Early:** If you can't deliver, say so ASAP
3. ✅ **Protect Deep Work:** Block time for complex tasks
4. ✅ **Learn to Say No:** Politely decline low-priority requests
5. ✅ **Review Daily:** Priorities change, adjust accordingly"

---

**78. What does "Definition of Done" mean? Why is it important?**

- **Answer:** "Definition of Done (DoD) is a checklist of criteria that must be met before a user story is considered complete.

**Purpose:**

- Creates shared understanding of "done"
- Ensures consistent quality
- Prevents incomplete work
- Reduces technical debt

**Example Definition of Done:**

```markdown
## Definition of Done

A user story is "Done" when ALL of the following are complete:

### Code Quality
- [ ] Code follows project style guide (ESLint passes)
- [ ] No console.log statements in production code
- [ ] No commented-out code
- [ ] TypeScript types properly defined (no `any`)
- [ ] Error handling implemented
- [ ] Input validation implemented

### Testing
- [ ] Unit tests written and passing (> 80% coverage)
- [ ] Integration tests written (if applicable)
- [ ] All existing tests still pass
- [ ] Edge cases tested
- [ ] Manual testing completed

### Code Review
- [ ] Pull request created with clear description
- [ ] At least one team member approved
- [ ] All review comments addressed
- [ ] CI/CD pipeline passes (tests, linting, build)

### Documentation
- [ ] Code comments added for complex logic
- [ ] API documentation updated (if API changes)
- [ ] README updated (if setup changes)
- [ ] CHANGELOG updated

### Deployment
- [ ] Merged to develop branch
- [ ] Deployed to staging environment
- [ ] Tested in staging by QA/Product Owner
- [ ] Ready for production deployment

### Business Acceptance
- [ ] All acceptance criteria met
- [ ] Product Owner reviewed and approved
- [ ] No known bugs
```

**Example User Story with DoD:**

```markdown
## User Story
As a school administrator, I want to upload student data via CSV so that I can quickly onboard many students.

## Acceptance Criteria
1. Administrator can upload a .csv file
2. System validates email format
3. System shows success message if upload succeeds
4. System shows clear error messages if validation fails
5. Database is updated with new student records

## Definition of Done Checklist
✅ Code Quality
  ✅ ESLint passes
  ✅ TypeScript strict mode enabled
  ✅ Error handling for invalid CSV format

✅ Testing
  ✅ Unit tests for CSV parser (15 tests)
  ✅ Integration test for upload endpoint
  ✅ Tested with 1000-row CSV file
  ✅ Tested edge cases: empty file, invalid format, duplicate emails

✅ Code Review
  ✅ PR #123 created
  ✅ Approved by @developer2
  ✅ All comments addressed
  ✅ CI pipeline passed

✅ Documentation
  ✅ API endpoint documented in README
  ✅ CSV format example provided
  ✅ Error codes documented

✅ Deployment
  ✅ Merged to develop
  ✅ Deployed to staging
  ✅ QA tested and approved
  ✅ Ready for prod deployment Friday

✅ Business Acceptance
  ✅ All acceptance criteria met
  ✅ Product Owner demo completed
  ✅ No known bugs

STATUS: ✅ DONE
```

**Why DoD is Important:**

**1. Prevents "Almost Done" Syndrome**

❌ Without DoD:
- "The feature works!" (but no tests, no documentation, full of bugs)

✅ With DoD:
- "The feature is complete, tested, reviewed, and deployed"

**2. Reduces Technical Debt**

Without DoD, teams often:
- Skip tests ("we'll add them later" → never happens)
- Skip documentation
- Deploy buggy code

With DoD, quality is built in from the start.

**3. Improves Velocity Predictability**

If "done" means different things, velocity estimates are meaningless.

Sprint 1: "Done" = code written (not tested) → 20 points
Sprint 2: "Done" = code + tests + review → 10 points

Which sprint was more productive? Hard to tell!

**4. Enables Continuous Deployment**

With DoD, every story is:
- Tested
- Reviewed
- Documented
- Production-ready

This enables safe, frequent deployments.

**DoD Levels:**

**Level 1: Story DoD** (for each user story)
- Code complete
- Tests passing
- Reviewed

**Level 2: Sprint DoD** (for each sprint)
- All stories meet Story DoD
- Deployed to staging
- Product Owner approved

**Level 3: Release DoD** (for production release)
- All sprints meet Sprint DoD
- Performance tested
- Security audit passed
- Documentation complete

**Common Mistakes:**

❌ DoD too strict (nobody can finish stories)
❌ DoD too loose (quality suffers)
❌ DoD not enforced (becomes meaningless)
❌ DoD never updated (becomes outdated)

**In this project:**

If we applied DoD to the CSV upload feature:

```
Definition of Done:

✅ Functionality
  ✅ Parses CSV with headers
  ✅ Validates email format
  ✅ Handles errors gracefully
  ✅ Saves to database

✅ Tests
  ✅ Unit tests: CSVParser.test.ts (12 tests)
  ✅ Service tests: DataUploadService.test.ts (8 tests)
  ✅ Integration: Upload endpoint test (3 tests)
  ✅ Coverage: 85%

✅ Code Quality
  ✅ TypeScript strict: true
  ✅ ESLint: 0 errors
  ✅ No console.log

✅ Review
  ✅ PR approved by 2 team members
  ✅ CI passed

✅ Documentation
  ✅ CSV format documented
  ✅ API endpoint in README

✅ Deployment
  ✅ Deployed to staging
  ✅ Manual test passed
  ✅ PM approved

STATUS: ✅ DONE → Ready for production!
```

**Interview Tip:** Having a clear Definition of Done prevents scope creep and ensures consistent quality across the team."

---
