---
title: "Artifactory Upgrade"
---

# Artifactory Upgrade

Our organization relied on a single self-hosted JFrog Artifactory instance as a repository for development artifacts, including container images, nuget packages, helm charts and other binary artifacts. The artifactory repo hosted nearly 2TB of data. As is common in growing organizations, ownership of artifactory was not well-defined. The primary users were developers, but they did not have the skills (or interest) to maintain it. The SRE team was the de-facto owner, but due to a series of events the original SMEs were no longer with the company. This created what is best described as a "situationship": artifactory was in desperate need of maintenance, but no one had the skill or motivation to do the needful. However, our hand was finally forced when we received an email from JFrog informing us that support for version 6 of Artifactory was ending in a few short weeks. Given how critical Artifactory was to our org, we had no choice but to upgrade to version 7 to ensure continued support. To add to the complexity, artifactory was hosted on a Windows VM. Managing windows VMs comes with added complexity, as there is normally second-class support for FOSS. I saw this as an opportunity to demonstrate my architecture, planning, and automation skills.

## Task

Upgrade artifactory to the most recent supported version to ensure continued support. I set the following project goals:

* Automate as much the upgrade process using a toolset that was familiar to most of our SRE team.
* Minimize downtime. If artifactory was down, developers could not build, push, or pull images. With a development team of approximately 100, this would grind development to a halt and be very expensive.
* No data loss.
* I wanted to test the upgrade process to ensure a high-level of confidence when it came time to perform the upgrade.
* I wanted to clearly communicate my progress with the wider development team to ensure no surprises come upgrade day.
* I also had to coordinate with the IAM team to ensure our SSO integration was successfully configured post-upgrade. Otherwise folks would not be able to login to the UI.

## Actions

After analyzing through the JFrog documentation upgrade guide, I came up with the following plan:

* Automation: Our org already had a sizeable chef and ansible footprint. I decided to use ansible because of the agentless architecture and ability to quickly iterate and test locally. To that end, I created a custom ansible playbook that performed 80% of the upgrade steps. By automating the process, it became self-documenting. This ensured that future upgrades
* Configured automated repository backups to ensure zero data loss. I also enabled de-duplication to ensure only the minimum required data was migrated to the new artifactory instance.
* Provisioned a separate virtual machine to test the playbook against. This gave me an opportunity to flesh out most of the bugs prior running the playbook against the production instance.
* Notified the development team of the upgrade window well in advance.
* I also took the opportunity to create a custom CNAME record to access artifactory. Prior to this, the artifactory URL was hard-coded to the DNS name of the virtual machine.
* I also took an opportunity to increase the JVM memory allocated to JFrog. Artifactory was hosted on a VM with 32GB of memory, but the JVM was capped at 6GB.

## Result

The upgrade was successful and occurred within the planned downtime window (~2.5 hrs). Post upgrade, I received feedback from the developers who reported the JFrog UI was noticeably faster, especially when viewing repos with potentially hundreds of artifacts. This was an unintended but welcome consequence of the upgrade.

### Tools & Skills

ansible, artifactory, planning, coordination, testing
