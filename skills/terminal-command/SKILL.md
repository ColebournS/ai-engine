---
name: terminal-command
description: Formats terminal commands for fast, safe, copy-paste-friendly execution. Use when providing shell commands, CLI instructions, terminal snippets, one-liners, or any bash/zsh/command-line operations to the user.
---

# Terminal Command

When providing terminal commands to the user, follow all rules below. These apply to every shell command in every code block, regardless of the tool or CLI involved.

## Rules

### 1. No Comments Inside Code Blocks

Never put comments (`#`, `//`, or otherwise) inside terminal code blocks. Comments break copy-paste workflows.

**Bad:**

```bash
# Get the instance ID
INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=my-app" --query 'Reservations[0].Instances[0].InstanceId' --output text)
# Stop the instance
aws ec2 stop-instances --instance-ids "$INSTANCE_ID"
```

**Good:**

Retrieve the instance ID and stop it:

```bash
INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=my-app" --query 'Reservations[0].Instances[0].InstanceId' --output text) &&
aws ec2 stop-instances --instance-ids "$INSTANCE_ID"
```

Put explanations in the surrounding prose, never inside the code block.

### 2. No Placeholders — Use Variables

Never use placeholders like `<your-arn-here>` or `INSERT_ID`. Instead, chain a lookup command that captures the value into a variable, then use that variable.

**Bad:**

```bash
aws ecs update-service --cluster my-cluster --service <INSERT_SERVICE_NAME> --desired-count 2
```

**Good:**

```bash
SERVICE_NAME=$(aws ecs list-services --cluster my-cluster --query 'serviceArns[0]' --output text | awk -F'/' '{print $NF}') &&
aws ecs update-service --cluster my-cluster --service "$SERVICE_NAME" --desired-count 2
```

If a value genuinely cannot be derived programmatically (e.g., a user's personal preference or a brand-new name they must choose), ask the user for the value conversationally before presenting the command. Then substitute it directly into the command — still no placeholders.

### 3. Chain Commands with &&

Where commands are logically sequential, chain them with `&&` so the user can copy-paste and run in one go. Break chains at natural boundaries (separate code blocks) when the user needs to inspect output before proceeding.

**Good — single copy-paste block:**

```bash
terraform init &&
terraform plan -out=tfplan &&
terraform apply tfplan
```

**Good — split when user should review between steps:**

Review the plan first:

```bash
terraform init && terraform plan -out=tfplan
```

Then apply:

```bash
terraform apply tfplan
```

### 4. Break Long Commands for Readability

When a command exceeds ~120 characters, use `\` line continuations to keep it readable. Align continuation lines for visual clarity.

**Bad:**

```bash
aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t3.medium --key-name my-key --security-group-ids sg-0123456789abcdef0 --subnet-id subnet-0123456789abcdef0 --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-instance}]'
```

**Good:**

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=my-instance}]'
```

### 5. Security — Environment-Aware Caution

Before providing any command that **modifies, deletes, or could disrupt resources**, assess the environment:

- **Production**: Always ask the user to confirm before providing destructive or irreversible commands. Flag the risk clearly and explain what will happen.
- **Staging/Dev**: Provide the commands but note any risks worth knowing about.
- **Read-only commands** (describe, list, get, status, logs): Provide freely in any environment without asking.

When a command supports `--dry-run` or an equivalent preview mode, suggest running it first so the user can verify before committing.

Destructive signals to watch for: `delete`, `destroy`, `terminate`, `rm -rf`, `drop`, `force`, `--no-dry-run`, `--force`, `purge`, `reset`, scaling to zero, modifying IAM policies, changing security groups, database migrations on production.

When concerned, present the situation to the user and wait for confirmation before giving the command.

### 6. Auto-Wait Instead of "Run Again"

Never tell the user to "keep running this command until the status changes." Instead, provide a command that waits, polls, or watches automatically.

**Bad:** "Run this command periodically until the status shows `active`"

**Good — use built-in waiters when available:**

```bash
aws ecs services wait services-stable --cluster my-cluster --services my-service
```

**Good — poll loop when no built-in waiter exists:**

```bash
while true; do
  STATUS=$(aws ecs describe-services --cluster my-cluster --services my-service --query 'services[0].status' --output text)
  echo "Current status: $STATUS"
  [ "$STATUS" = "ACTIVE" ] && break
  sleep 10
done
```

This applies broadly — any scenario where the user would otherwise manually repeat a command to check for a condition.

Prefer built-in wait/watch mechanisms when the tool provides them (e.g., `aws wait`, `kubectl rollout status`, `docker wait`). Fall back to a poll loop with `sleep` and a clear exit condition when no built-in option exists.

Include a timeout safeguard for poll loops when the wait could be unbounded:

```bash
TIMEOUT=300
ELAPSED=0
while true; do
  STATUS=$(some-command --query 'status' --output text)
  echo "Status: $STATUS (${ELAPSED}s elapsed)"
  [ "$STATUS" = "READY" ] && break
  [ "$ELAPSED" -ge "$TIMEOUT" ] && echo "Timed out after ${TIMEOUT}s" && exit 1
  sleep 10
  ELAPSED=$((ELAPSED + 10))
done
```

## Validation

Before presenting a command block, verify:

- No comments inside the code block
- No placeholders — all values derived or pre-substituted
- Sequential commands chained with `&&`
- Long commands broken with `\` continuations
- Destructive commands flagged for the target environment
- Polling/waiting is automated, not manual

## Error Handling

If you cannot derive a value programmatically for Rule 2:
→ Ask the user for the value in prose, then construct the full command with no placeholders.

If you are unsure whether a command is destructive:
→ Err on the side of caution — flag it and ask.

If you are unsure which environment the user is targeting:
→ Ask before providing any commands that modify resources.

If a built-in waiter or watch command does not exist for Rule 6:
→ Use a poll loop with sleep, a clear exit condition, and a timeout.
