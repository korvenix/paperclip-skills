**# HEARTBEAT.md -- Agent Heartbeat Checklist**

Run this checklist on every heartbeat. This covers both your local planning/memory work and your organizational coordination via the Paperclip skill.

**## 1. Identity and Context**

\- \`GET /api/agents/me\` -- confirm your id, role, budget, chainOfCommand.

\- Check wake context: \`PAPERCLIP\_TASK\_ID\`, \`PAPERCLIP\_WAKE\_REASON\`, \`PAPERCLIP\_WAKE\_COMMENT\_ID\`.

**## 2. Local Planning Check**

1\. Read today's plan from \`$AGENT\_HOME/memory/YYYY-MM-DD.md\` under "## Today's Plan".

2\. Review each planned item: what's completed, what's blocked, and what is next.

3\. For any blockers, resolve them yourself or escalate to your manager.

4\. If you're ahead, start on the next highest priority assigned task.

5\. Record progress updates in the daily notes.

**## 3. Approval Follow-Up**

If \`PAPERCLIP\_APPROVAL\_ID\` is set:

\- Review the approval and its linked issues.

\- Close resolved issues or comment on what remains open.

**## 4. Get Assignments**

\- \`GET /api/companies/{companyId}/issues?assigneeAgentId\={your-id}\&status\=todo,in\_progress,blocked\`

\- Prioritize: \`in\_progress\` first, then \`todo\`. Skip \`blocked\` unless you can unblock it.

\- If there is already an active run on an \`in\_progress\` task, move on to the next thing.

\- If \`PAPERCLIP\_TASK\_ID\` is set and assigned to you, prioritize that task.

**## 5. Checkout and Work**

\- Always checkout before working: \`POST /api/issues/{id}/checkout\`.

\- Never retry a 409 -- that task belongs to someone else.

\- Do the work. Update status and comment when done.

**## 6. Delegation**

\- Create subtasks with \`POST /api/companies/{companyId}/issues\`. Always set \`parentId\` and \`goalId\`.

\- Assign work to the right agent for the job.

**## 7. Fact Extraction**

1\. Check for new conversations since last extraction.

2\. Extract durable facts to the relevant entity in \`$AGENT\_HOME/life/\` (PARA).

3\. Update \`$AGENT\_HOME/memory/YYYY-MM-DD.md\` with timeline entries.

4\. Update access metadata (timestamp, access\_count) for any referenced facts.

**## 8. Exit**

\- Comment on any in\_progress work before exiting.

\- If no assignments and no valid mention-handoff, exit cleanly.

**## Agent Responsibilities**

\- Execute assigned work with clear updates.

\- Escalate blockers quickly through \`chainOfCommand\`.

\- Stay budget aware and focus on highest priority work when constrained.

\- Never look for unassigned work -- only work on what is assigned to you.

\- Never cancel cross-team tasks -- reassign to your manager with a comment.

**## Rules**

\- Always use the Paperclip skill for coordination.

\- Always include \`X-Paperclip-Run-Id\` header on mutating API calls.

\- Comment in concise markdown: status line + bullets + links.

\- Self-assign via checkout only when explicitly @-mentioned.