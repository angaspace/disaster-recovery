# disaster-recovery

Backup, restore and failover playbooks for Anga Space client environments.

## Playbooks
| Playbook | Scenario | Tested |
| :--- | :--- | :--- |
| `runbooks/ebs-snapshot-failover.md` | EC2 or storage failure in one AZ, restore from snapshot into a healthy AZ | Not yet |

## Principles
- A backup is only real once you have restored from it.
- Targets (RTO/RPO) are measured in drills, not assumed.
- Practice in a sandbox before any client environment.

## Planned
- Automated snapshot schedule (AWS Data Lifecycle Manager)
- Restore test script
- Cross-region copy runbook
