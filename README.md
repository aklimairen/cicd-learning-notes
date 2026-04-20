# CI/CD Learning Notes

This repo is my personal notes from the [Continuous Integration and Continuous Delivery (CI/CD)](https://www.coursera.org/learn/continuous-integration-and-continuous-delivery-ci-cd) course by IBM on Coursera. I structured everything as I went through each topic so I could actually refer back to it — not just watch the videos and forget.

The notes are written in my own words, with real examples and commands I ran during the labs. If something confused me, I tried to write it in a way that would make sense to me later.

The final hands-on project from this course lives in a separate repo → **[ci-cd-final-project](https://github.com/aklimairen/ci-cd-final-project)**

---

## What's covered

| #  | Topic                                                                | What I learned                                                        |
| -- | -------------------------------------------------------------------- | --------------------------------------------------------------------- |
| 01 | [CI/CD Basics](./01_basics/what-is-cicd.md)        | What CI and CD actually mean, the DevOps pipeline, IaC                |
| 02 | [GitHub Actions](./02_github_actions/workflows.md) | Events, runners, jobs, steps, actions, workflow YAML                  |
| 03 | [Tekton](./03_tekton/tekton-intro.md)              | Tasks, Pipelines, Triggers, Tekton Hub, image building                |
| 04 | [OpenShift](./05_OpenShift/openShift-note.md)      | Projects, ImageStreams, Routes,`oc`CLI, SCCs, deploying from Tekton |

---

## How I structured the notes

Each topic follows the same format — a short overview, the core concepts, a real code/command example, and bullet points for the things worth remembering. I kept it short on purpose. Long notes don't get read.

---

## Tools & stack covered

* **GitHub Actions** — CI workflows, secrets, matrix builds, artifacts
* **Tekton** — Kubernetes-native pipelines, tasks, triggers
* **OpenShift** — Deployment, routing, security context, `oc` CLI
* **Docker** — Containerization, Dockerfiles, image registries
* **Python** — Flask microservice used throughout the labs

---

## Final project

Everything I practiced in these notes came together in the capstone project — a full CI/CD pipeline around a Python/Flask microservice:

* GitHub Actions handles **CI** (lint + test on every push)
* Tekton Pipelines on OpenShift handles **CD** (cleanup + test inside the cluster)

→ [ci-cd-final-project](https://github.com/aklimairen/ci-cd-final-project)
