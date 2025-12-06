# Section 2: Argo CD Architecture & Concepts

## Overview

This section provides a deep dive into how Argo CD works under the hood. Understanding the architecture and core concepts is essential for effective troubleshooting and advanced use cases.

## Learning Objectives

By the end of this section, you will:
- Understand Argo CD's component architecture
- Know the difference between Applications, Projects, and Repositories
- Understand desired vs live state
- Master sync, health, and drift concepts

## Lectures

1. [2.1 Argo CD Components (API Server, Repo Server, Controller)](./lecture-2.1.md)
2. [2.2 Apps, Projects, Repos](./lecture-2.2.md)
3. [2.3 Desired vs Live State](./lecture-2.3.md)
4. [2.4 Sync, Health, and Drift](./lecture-2.4.md)

## Key Takeaways

- Argo CD consists of three main components: API Server, Repo Server, and Application Controller
- Applications define what to deploy and where
- Projects provide boundaries for multi-tenancy
- Argo CD continuously compares desired (Git) vs live (cluster) state
- Sync status and health status are independent concepts

## Next Steps

After completing this section, proceed to [Section 3: Lab Setup](../section-03/README.md) to set up your environment for hands-on practice.

## Quiz

Complete the [Section 2 Quiz](../../quizzes/section-02-quiz.md) to test your understanding.
