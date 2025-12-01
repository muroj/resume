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

# Kubernetes Upgrade (v1.14 → v1.23)

## Situation

Our kubernetes clusters were running [version 1.14](https://kubernetes.io/blog/2019/03/25/kubernetes-1-14-release-announcement/), which at the time was at three years old and no longer eligible for support. The clusters were provisioned years prior using a tool called [kops](https://kops.sigs.k8s.io/) by an engineer who was no longer with the company. The kops version used to provision the clusters was also years old and running a deprecated version. No one was familiar with kops and no one dared upgrade the clusters. However, our hand was forced when our production cluster could no longer scale out due to worker nodes failing to join the cluster due to a deprecated dependency in the kops script. The inability to scale out put our customers at risk, as our login service was often running at capacity, and we often needed to increase the deployment size to prevent failed login attempts.

## Task

* Create a plan to upgrade our clusters to the oldest support version. I targeted the oldest version that was still eligible for support rather than the most recent version to minimize application changes. I wanted to limit the risk of breaking anything at the application layer. I was familiar with our helm charts, so I could work with my direct team to PR any required manifest changes. 
* Minimize downtime. The SRE team did not have it's own cluster. We shared the dev cluster wit the app developers. I could not break this cluster because it would block the entire development team. In other words, I had to get it right the first time.  

## Actions

* I started with the development cluster. The first step was to familiarize myself with kops by reading the documentation. Kops uses YAML files to define the desired cluster state. I located our YAML files, which had become woefully unmaintained, and reconiled the YAML config with the live cluster state. After doing so, I could then proceed with the upgrade process.
* I performed a multi-stage upgrade in our dev cluster. I could not upgrade directly to version 1.23, I first had to upgrade to 1.17. To do that, I had to upgrade kops to a version that supported the desired k8s version. 
* I meticulously documented the entire process to allow.
* I ran every command with the dry-run option to allow me to review the diff in the cluster state.
* I performed the upgrade during off-hours to minimize downtime for the developers.
* I verified that all deployments and services running in the cluster were valid.

## Result

The upgrade was successful. The end result was the cluster was running a supported version. This removed deprecated APIs and ensured our customer workload was running on a supported version. It also fixed the worker node issue preventing the cluster from scaling out.

# Implemented PodPriorities to Remove Manual Operator Intervention

## Situation

Our software release cycle usually triggered a restart of all kubernetes Deployments and StatefulSets to pick up the latest container image version. This restart would normally result in several critical pods getting stuck in NotReady state due to insufficient resources on the worker nodes, even though there was sufficient capacity on the whole cluster. but the scheduler did not have enough context to ensure all pods would get scheduled. Resolving this would require manual operator intervention: an SRE would manually kill some non-critical pods to allow the critical pods to be scheduled. For any computer science nerds, this was an instance of the [bin-packing problem.](https://en.wikipedia.org/wiki/Bin_packing_problem).

## Task

Automate the manual steps taken by the operator to resolve the pod scheduling issue.

## Actions

I used PodPriority and Preemption policies to provide the scheduler with enough context to resolve the scheduling issues. To do this, I worked with the development team to classify each service into an appropriate priority level: high, medium or low. Services with high priority, such as user login or pods serving the web front end, could preempt lower priority services. I tested the eviction behavior to ensure it functioned as we intended.

## Result 

Reduced manual operator toil. Captured the desired application state in code.  Improved release reliability and reduced deployment delays.

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

# SCDP Unused VMs