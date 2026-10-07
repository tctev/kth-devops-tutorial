# Guide for the Argo CD Tutorial

This document explains the structure of the project and how the current
Killercoda scenario works. It is mainly intended to help with the
writing, presentation, and pedagogical flow of the tutorial.

## Useful links

-   **Killercoda tutorial:**
    https://killercoda.com/s-riviere/scenario/gitops-argocd
-   **Git repository:** https://github.com/s-riviere/kth-devops-tutorial

The tutorial is located in the `gitops-argocd/` directory of the
repository.

------------------------------------------------------------------------

## Project structure

``` text
gitops-argocd/
├── index.json
├── background.sh
├── foreground.sh
├── application.yaml
├── intro/
│   └── text.md
├── step1/
│   ├── text.md
│   └── verify.sh
├── step2/
│   ├── text.md
│   └── verify.sh
├── step3/
│   ├── text.md
│   └── verify.sh
├── step4/
│   ├── text.md
│   └── verify.sh
├── gitops/
│   └── app/
└── finish/
    └── text.md
```

### `index.json`

This is the main Killercoda configuration file.

It defines:

-   the tutorial title and description;
-   the introduction;
-   the list and order of steps;
-   the text file used by each step;
-   the verification script used by each step;
-   the Kubernetes environment used by the scenario.

The order in `details.steps` is the order in which the steps appear in
Killercoda.

### `intro/text.md`

The introduction displayed before the steps.

### `stepX/text.md`

The content displayed to the learner for a given step.

This is where the tutorial instructions, explanations, commands,
expected results, and educational content should be written.

### `stepX/verify.sh`

The automatic verification for a step.

When the learner clicks **CHECK**, Killercoda runs this script. The
script should exit with code `0` when the expected state has been
reached and with a non-zero code otherwise.

The verification scripts are therefore about **checking state**, not
explaining the concept.

### `finish/text.md`

The final page/conclusion of the tutorial.

------------------------------------------------------------------------

## What the tutorial currently does

The technical goal is to demonstrate the core GitOps workflow with Argo
CD:

> Git contains the desired state. Argo CD continuously compares that
> desired state with the actual Kubernetes state and can automatically
> correct drift.

The current scenario goes through four steps:

1.  **Check the Kubernetes cluster**
    -   Kubernetes is already prepared by the environment.
    -   Argo CD is installed automatically in the background.
2.  **Open Argo CD**
    -   The learner accesses the Argo CD UI.
3.  **Deploy the GitOps application**
    -   The learner applies the Argo CD `Application`.
    -   Argo CD uses the Git repository as the source of truth.
    -   The application is synchronized and becomes `Synced / Healthy`.
4.  **Create drift and observe self-healing**
    -   The learner deliberately deletes a Kubernetes resource.
    -   The actual cluster state no longer matches the desired state in
        Git.
    -   Argo CD detects the drift and recreates the resource
        automatically.

### Important: the current steps are not the intended final scope

The current four-step flow should be considered a **technical
baseline**, not the final ambition or content of the tutorial.

The implementation has intentionally focused first on getting the code,
infrastructure, Argo CD setup, GitOps application, and self-healing
demonstration working.

The next work is mainly on:

-   improving the pedagogical progression;
-   deciding what should be explained and when;
-   making the instructions clearer;
-   improving the learner experience;
-   adding context around the commands;
-   deciding whether additional GitOps / Argo CD concepts should be
    introduced.

So the current scenario should be viewed as a working foundation that
can be reorganized or expanded from the writing/pedagogy side.

------------------------------------------------------------------------

## Git is the source of truth

The Git repository used by the Argo CD `Application` is:

``` text
https://github.com/s-riviere/kth-devops-tutorial
```

More specifically, the GitOps application manifests are under the
`gitops-argocd/gitops/` directory.

This repository represents the **desired state** of the application.

The important distinction for the tutorial is:

``` text
Git repository
    │
    │ desired state
    ▼
  Argo CD
    │
    │ synchronization
    ▼
 Kubernetes cluster
    │
    │ actual state
    └───────────────┐
                    │
              drift detected
                    │
                    ▼
                 Argo CD
                    │
                    └──► self-healing
```

The learner should therefore understand that Kubernetes is not manually
maintained as the ultimate source of truth. The desired configuration
lives in Git, and Argo CD is responsible for keeping the cluster aligned
with it.

------------------------------------------------------------------------

## `application.yaml`

`application.yaml` is the Argo CD `Application` manifest used in Step 3.

It tells Argo CD where the Git source is and where/how the application
should be deployed.

The manifest itself is prepared automatically by `background.sh` and
stored in:

``` text
/tmp/my-app-application.yaml
```

The learner applies it during Step 3:

``` bash
kubectl apply -f /tmp/my-app-application.yaml
```

The important point is that **preparing the `Application` and actually
applying it are separate actions**. The background setup prepares the
environment, while Step 3 is part of the learner's GitOps workflow.

------------------------------------------------------------------------

## `gitops/`

The `gitops/` directory contains the Kubernetes manifests that represent
the desired state of the application.

These files are stored in the Git repository and are the configuration
Argo CD continuously compares against the actual cluster.

For example, if the Git repository says that a Service should exist and
the learner deletes that Service manually, the cluster has drifted from
the desired state. Argo CD can then recreate it.

------------------------------------------------------------------------

## `background.sh`

`background.sh` prepares the infrastructure automatically so that the
learner can focus on GitOps rather than Kubernetes/Argo CD installation
details.

It currently:

-   waits for Kubernetes to become ready;
-   creates the `argocd` namespace;
-   installs a pinned Argo CD version;
-   configures Argo CD;
-   restarts the relevant Argo CD components;
-   waits for the Argo CD admin credentials to exist;
-   downloads the `Application` manifest;
-   starts the Argo CD port-forward;
-   waits until the Argo CD API is accessible;
-   creates `/tmp/argocd-setup-complete` when setup is finished.

The background script **does not deploy the GitOps application**. That
happens explicitly in Step 3.

This separation is intentional: installation and infrastructure
preparation happen automatically, while the GitOps workflow remains part
of the learner's hands-on experience.

------------------------------------------------------------------------

## `foreground.sh`

`foreground.sh` provides feedback while the background setup is running.

An important Killercoda detail is that the foreground content is
injected into the learner's terminal. It should therefore be kept simple
and user-friendly rather than treated like a normal hidden
initialization script.

It waits for:

``` text
/tmp/argocd-setup-complete
```

and displays the current setup status written by `background.sh`, for
example:

``` text
⏳ Installing Argo CD...
⏳ Starting the Argo CD web endpoint...
✅ Argo CD is ready
```

Its purpose is mainly **user feedback**, not environment setup.

------------------------------------------------------------------------

## Overall division of responsibilities

A useful way to think about the project is:

``` text
index.json
    → scenario structure

text.md files
    → what the learner reads and does

verify.sh files
    → automatic checks

background.sh
    → infrastructure / Argo CD preparation

foreground.sh
    → setup feedback in the terminal

application.yaml
    → tells Argo CD what Git source to use

gitops/
    → desired application state stored in Git
```

The code and environment setup are currently in place. The main
remaining work is to turn that technical foundation into a clear,
well-paced learning experience.
