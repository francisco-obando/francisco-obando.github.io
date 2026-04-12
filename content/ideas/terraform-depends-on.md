---
title: "When to Use Explicit 'depends_on' in Terraform"
date: 2026-02-15T12:00:00Z
summary: "A lesson in why implicit dependencies aren't always enough, especially when managing NLB targets for existing databases."
---

## The Trap of Implicit Dependencies

Terraform is excellent at building a dependency graph based on resource attributes (e.g., passing a database ID to a security group). However, there are "silent" failure modes where the dependency graph looks correct in a `terraform plan`, but the execution order causes service degradation.

### The Real-World Scenario: NLBs and Databases

I recently encountered a case while running data scripts to add a **Network Load Balancer (NLB)** to an existing set of databases. 

**The Setup:**
1. Databases are already provisioned.
2. A new NLB is added to front these databases.
3. NLB Target Group Attachments use the database IP/IDs.

**The Failure:**
When the databases were modified (e.g., a scaling event or a configuration change), Terraform didn't recognize that the **NLB Targets** needed to be refreshed *after* the database modification was fully complete. Because there was no direct attribute dependency that triggered a "replace" on the attachment, the NLB continued to point to "stale" or transitioning database instances.

**The Result:**
The `terraform plan` looked perfectly valid, but the lack of an explicit dependency caused the NLB to lose its connection to the healthy database nodes, leading to immediate service degradation.

### The Rule of Thumb

Don't rely solely on implicit attribute links when:
1. **Networking logic is decoupled**: Resources like NLBs, Listeners, and Target Groups often have "loose" connections to the underlying compute/database.
2. **Third-party scripts are involved**: If you are running `null_resource` or data scripts that interact with the infrastructure.
3. **State doesn't capture the "Ready" signal**: Just because a database resource is "updated" in AWS doesn't mean it's ready to accept traffic on a specific port.

**The Fix:**
Always add an explicit `depends_on` block in your `aws_lb_target_group_attachment` or similar resources to ensure they are the very last thing to be touched after a backend resource modification.

```hcl
resource "aws_lb_target_group_attachment" "db_attachment" {
  target_group_arn = aws_lb_target_group.db_tg.arn
  target_id        = aws_db_instance.primary.id
  port             = 5432

  # Explicitly wait for the DB to be fully ready/modified
  depends_on = [aws_db_instance.primary]
}
```
