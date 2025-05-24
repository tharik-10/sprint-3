
# **Declarative Jenkins Pipeline for Commit Sign-off**
| Created        | Last updated      | Version         | author|  Internal Reviewer | L0 | L1 | L2|
|----------------|----------------|-----------------|-----------------|-----|------|----|----|
| 2025-05-24  | 2025-05-24   |     Version 1         |  Mohamed Tharik |Priyanshu|Khushi|Mukul Joshi |Piyush Upadhyay|

## Table of Contents 

## Introduction 
Commit sign-off is a lightweight certification that ensures contributors are submitting code they have the right to share, often used in open-source and enterprise environments to comply with the Developer Certificate of Origin (DCO). This document outlines how to create a Declarative Jenkins Pipeline that validates commit sign-offs and runs generic CI checks, helping teams automate compliance and maintain code quality across all contributions.

## Prerequisites

| Requirement                  | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Jenkins Server              | A running Jenkins instance with proper access rights                        |
| Git Repository              | Source code repository (e.g., GitHub, GitLab) with commit history           |
| Pipeline Plugin             | Jenkins Pipeline plugin installed                                           |
| Git Installation            | Git should be installed and available on Jenkins agents                    |
| SCM Integration             | Jenkins configured with credentials to access the Git repository            |
| Basic Git Knowledge         | Understanding of git commits and `Signed-off-by` convention                 |
| Jenkins Agent (Optional)    | A configured Jenkins agent for scalable build execution                     |

## Steps to Setup Declarative Pipeline for Commit Sign-off

## Best Practices
| Best Practice                            | Explanation                                                                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Always check against a base branch** | Use `origin/main` or `origin/master` as the base for comparing new commits                                                           |
| **Fail early**                         | Catch missing sign-offs before running full CI to save resources                                                                     |
| **Display clear messages**             | Make it easy for contributors to understand what failed and how to fix it                                                            |
| **Enforce sign-off via hooks**         | Optionally enforce with pre-commit hooks or GitHub branch protection rules                                                           |
| **Document contribution guidelines**   | Include DCO and sign-off process in `CONTRIBUTING.md`                                                                                |

## Conclusion
Implementing a Declarative Jenkins Pipeline with Commit Sign-off validation is a robust step toward securing your CI/CD processes. It ensures contributions are legitimate and traceable while keeping pipelines clean, standardized, and automated. Combining this with automated CI checks strengthens your development lifecycle by promoting responsibility, legal clarity, and quality assurance.

## Contact Information
| Name | Email address         |
|------|------------------------|
| Mohamed Tharik  | md.tharik.sanaatak@mygurukulam.co    |

## References

| Link | Description |
|------|-------------|
