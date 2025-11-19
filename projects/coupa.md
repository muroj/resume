STAR Method:

Situation: Briefly describe the context of the situation you were in. Provide enough background so the interviewer understands the scenario.
Task: Explain the goal you were trying to achieve or the challenge you needed to overcome.
Action: Detail the specific steps you took to address the situation or task. Be specific about your individual contributions.
Result: Explain the outcome of your actions. Include the results of your work and any lessons you learned. It is often helpful to quantify your results if possible

# Automated Artifactory Upgrade (v6 → v7) with Ansible

Situation: Coupa relied on a single self-hosted JFrog Artifactory instance as a repository for development artifacts, including container images, nuget packages, helm charts and other binary artifacts. The artifacty repo hosted close to 2TB of data. As is common in growing organizaitons, ownership of this Artifactory instance was not clearly defined. This led to a situation where the Artifactory was in sore need of maintenance. It was at version 6, which was no longer eligible for support from JFrog. As the sole artifact repo hosting production artifacts, we had no choice but to upgrade to ensure continued support of this critical peice of infrastructure. Unfortunately, no one on the team had experience administering Artifactory, including myself. To add to that complexity, the artifactory was hosted on a Windows VM. Managing windows VMs comes with added complexity, as there is normally second-class support for FOSS. I saw this as an opportunity to demonstrate my architecture, planning, and automation skills. 

## Task 

The task was to upgrade artifactory to the most recent supported version to ensure continued support. I set the following project goals:

* Automate as much the upgrade process using a toolset that was familiar to most of our SRE team.
* Minimze downtime. If artifactory was down, developers could not build, push, or pull images. With a development team of approximately 100, this would grind development to a halt and be very expensive.
* No data loss. 
* I wanted to test the upgrade process to ensure a high-level of confidence when it came time to perform the upgrade.
* I wanted to clearly communicate my progress with the wider development team to ensure no surprises come upgrade day.
* I also had to coordinate with the IAM team to ensure our SSO integration was successfully configured post-upgrade. Otherwise folks would not be able to login to the UI.

## Actions

After analyzing through the JFrog documentation upgrade guide, I came up with the following plan:

* Automation: Coupa already had a sizeable chef and ansible footprint. I decided to use ansible because of it's agentless architecture and ability to quickly iterate and test locally. To that end, I created a custom ansible playbook that peformed 80% of the upgrade steps. By automating the process, it became self-documenting. This ensured that future upgrades 
* Configured automated repository backups to ensure zero data loss. I also enabled de-duplication to ensure only the minimum required data was migrated to the new artifactory instance.
* Provisioned a separate virtual machine to test the playbook against. This gave me an opportunity to flesh out most of the bugs prior running the playbook against the production instance.
* Notified the development team of the upgrade window well in advance.
* I also took the opportunity to create a custom CNAME record to access artifactory. Prior to this, the artifactory URL was hard-coded to the DNS name of the virtual machine. 
* I also took an opportunity to increase the JVM memory allocated to JFrog. Artifactory was hosted on a VM with 32GB of memory, but the JVM was capped at 6GB.  

## Result

The upgrade was successful and occurred within the planned downtime window (~2.5 hrs). Post upgrade, I received feedback from the developers who reported the JFrog UI was noticeabley faster, especially when viewing repos with potentially hundreds of artifacts. This was an unintended but welcome consequence of the upgrade. 

Why this works: Shows automation, repeatability, and configuration management maturity—core platform-engineering signals.

# Sole Owner of Kubernetes Upgrade (v1.14 → v1.23)

Problem: The cluster was stuck on an unsupported version and lacked modern API stability, security patches, and scheduling features.
Actions:

Planned and executed the full multi-version upgrade path with zero data loss.

Resolved deprecated APIs, CRD breakages, and controller-manager compatibility issues.

Coordinated component upgrades (CNI, kube-proxy, ingress) and ran canary workloads to validate behavior.
Impact:

Brought the platform back into a supported state and enabled adoption of newer k8s features.

Reduced operational risk and improved cluster stability for all services.

Why this works: Communicates end-to-end ownership, deep Kubernetes competence, and real production responsibility.

# Implemented PodPriorities to Remove Manual Operator Intervention

Problem: Release cycles required manual scheduling tweaks to ensure critical workloads were deployed before lower-priority ones.
Actions:

Designed and implemented PodPriority and Preemption policies across namespaces.

Identified critical workloads and mapped them to appropriate priority classes.

Tested eviction behavior and validated disruption impact.
Impact:

Eliminated manual operator steps from the deployment process.

Improved release reliability and reduced deployment delays.

Why this works: Shows ability to modify cluster scheduling behavior and improve platform ergonomics.

# Rebuilt Machine Image Pipeline to Enable Developer Self-Service

Problem: The machine image build/promote pipeline was broken, blocking releases and forcing operators to manually push images across environments.
Actions:

Debugged and repaired image build automation and promotion logic.

Standardized artifact metadata and validation checks.

Exposed a safe, self-service interface for developers to promote images from dev → QA.
Impact:

Restored a critical CI/CD capability and eliminated a recurring operational bottleneck.

Reduced engineering wait times and improved release velocity.

Why this works: This is developer-experience work—very attractive to platform-engineering hiring managers.

# Upgraded PingFederate SSO by Stabilizing Fragile Chef Module

Problem: The existing Chef module was unreliable and regularly broke the PingFederate deployment pipeline.
Actions:

Reverse-engineered the failing cookbook logic and removed brittle code paths.

Added idempotency, proper service notifications, and validated upgrade ordering.

Performed integration tests to ensure sessions, certs, and cluster replication remained intact.
Impact:

Enabled a smooth SSO upgrade and reduced future deployment failures.

Increased reliability of a mission-critical authentication service.

Why this works: Shows configuration automation skill + ability to rescue fragile infra automation.

Owned and Modernized AWS ALB Terraform Module (v6 → v9)

# Problem: The team relied on an outdated Terraform ALB module that blocked new features and security improvements.
Actions:

Upgraded module configurations from v6 to v9, resolving breaking changes and new variable requirements.

Wrote a Python script using sh to automate Terraform plan/apply sequences and prevent operator error.

Improved state management and documented safe patterns for future upgrades.
Impact:

Brought ALB configurations to current standards and improved maintainability.

Reduced manual Terraform toil and increased deployment consistency.

Why this works: This positions you as someone who owns infrastructure-as-code modules—not someone who just “runs terraform.”

# Custom Terraform Provider

Built a custom Terraform provider to integrate internal platform APIs, replacing a manual, error-prone provisioning workflow with fully declarative Configuration-as-Code. Enabled teams to manage platform resources through version-controlled Terraform plans and eliminated inconsistent, operator-driven changes.
