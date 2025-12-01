## Azure VM Disk Backups

### Situation

SRE receives frequent, ad-hoc requests from various teams to create azure VM disk snapshots (i.e backups). These requests oftentimes take *weeks* to fulfill due to conflicting priorities between teams. Historically SREs would create the snapshot manually via the azure console or the CLI. The process is manual, time-consuming, and takes an unnecessarily long time to fulfill causing frustration with the requesting team. In summary, it is toil, and a great opportunity for automation.

### Task

Define an automated process to streamline the creation of  disk snapshot requests. The process had the following criteria:

* Self-service. Teams should not have to submit a ticket and wait weeks for SRE to priotize it.
* Automated: SREs should run ad-hoc CLI command to create a snapshot. With dozens of subscriptions and tens to hundres of resources groups, it is too easy to make a mistake.
* Minimal cost: Minimize storage costs by preventing the proliferation of unused snapshots

### Action

I created a set of terraform modules that allowed teams to self-service their own backups requests. The modules defined the following resources:

* A [backup vault](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_protection_backup_vault) per azure subscription
* One or more [backup policies](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_protection_backup_policy_disk)
* One or more [VM-based disk snapshots](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/data_protection_backup_instance_disk)

The solution utilized our existing GitOps pipeline to automate the creation of backup resources. I worked with a representative team (InfoSec) to test the solution to ensure it met their requirements. I also evangalized the solution to the wider SRE team to raise awareness and encourage adoption.

### Result

* Reduced friction between SRE and service teams. Teams no longer had to wait weeks for their disk snapshot request to get prioritized and fulfilled. 
* Disk snapshots were automated and scheduled. Backup policies supported different service levels depending on workload criticality. Stale snapshots are automatically purged based on a well-defined retention policy.

## AWS Migration

### Situation

### Task

### Action

### Result

## Azure Blob Storage Reduction (FinOps)

### Situation

### Task

### Action

### Result

## IP Metrics Application

### Situation

### Task

### Action

### Result

## Golang Ops CLI maintenance

## Golang Application Manifest Tool (Failed)

### Situation

### Task

### Action

### Result

## Golang Ops Web Service API 

### Situation

### Task

### Action

### Result