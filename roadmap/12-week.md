# Platform Engineering: A 12-Week Course

This course is designed for a Go backend engineer moving toward platform engineering. Each study day explains why the topic matters, what you will learn, gives you a hands-on task, and points to written reading material. There are no video assignments.

**Pace:** 5 study days per week, about 60–90 minutes per day (6–8 hours weekly). Treat each “day” as one study session; take rest days whenever needed. Use one Go HTTP API as the running example. Keep the work local until you intentionally choose a cloud provider.

## Course map

| Week | Theme | Milestone |
|---|---|---|
| 1 | Linux and networking | Explain how a request reaches a process and inspect it from the command line. |
| 2 | Containers | Run the Go API and PostgreSQL reproducibly with Docker Compose. |
| 3 | Git and CI | Automatically test and build the service on every pull request. |
| 4 | Production-ready Go service | Add configuration, health checks, graceful shutdown, and structured logs. |
| 5 | Kubernetes foundations | Deploy and reach the API in a local Kubernetes cluster. |
| 6 | Kubernetes operations | Configure, observe, roll out, roll back, and troubleshoot the workload. |
| 7 | Terraform | Describe and safely manage a small set of local resources as code. |
| 8 | Reusable deployment configuration | Package Kubernetes configuration for repeatable environments. |
| 9 | OpenTelemetry | Emit traces and metrics from the Go service. |
| 10 | Prometheus and dashboards | Query service health and build a useful dashboard. |
| 11 | Platform as a product | Create a simple golden path for starting and deploying a service. |
| 12 | Reliability and capstone | Practice SLO thinking and present the completed platform workflow. |

## Running project

Use a small Go HTTP API with a health endpoint and, optionally, PostgreSQL. Extend the same application through the course instead of starting over for each topic. Store each week's exercise in the matching folder under `labs/`; keep the final integrated project in `projects/go-api-platform-demo/`.

---

## Week 1 — Linux and networking

**Context:** Platform work often starts with a symptom such as “the service is down.” These sessions build a practical model of files, processes, ports, and HTTP so you can find where a request or process is failing.

### Day 1 — Shell, paths, and files

- **Learn:** Navigate a Linux filesystem, inspect files, and compose simple commands with pipes and redirection.
- **Practice:** Create a workspace, inspect the Go project, and save command output to a log file.
- **Read:** [Ubuntu command line for beginners](https://ubuntu.com/tutorials/command-line-for-beginners).

### Day 2 — Users, permissions, and processes

- **Learn:** How users, file permissions, process IDs, and signals affect a running service.
- **Practice:** Start the API, find its process, inspect it, send a graceful signal, and restart it.
- **Read:** [Ubuntu: Linux processes](https://ubuntu.com/server/docs/how-to/software/monitoring/processes/).

### Day 3 — Ports and TCP connections

- **Learn:** How clients connect to a process through an IP address and port; distinguish connection failures from application errors.
- **Practice:** Use `curl` and available socket tools to inspect the API's listener and test a closed port.
- **Read:** [Cloudflare Learning Center: What is TCP/IP?](https://www.cloudflare.com/learning/network-layer/what-is-tcp-ip/).

### Day 4 — DNS, HTTP, and TLS

- **Learn:** How DNS resolves a name, how HTTP requests and responses are structured, and what TLS protects.
- **Practice:** Inspect DNS lookup results and use `curl -v` against a local and a public HTTPS endpoint.
- **Read:** [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/), [MDN: An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview).

### Day 5 — Logs and first-response troubleshooting

- **Learn:** A repeatable first-response sequence: check process, listener, recent logs, and dependencies.
- **Practice:** Break the API's port or database address, diagnose the failure, and write down the commands that identified it.
- **Read:** [The Twelve-Factor App: Logs](https://12factor.net/logs).

**Milestone:** Explain the path from a DNS name to an HTTP handler and show how you would investigate a connection failure.

---

## Week 2 — Docker and Docker Compose

**Context:** Containers make an application's runtime reproducible. The aim is to understand the image, container, network, and volume model well enough to package and debug the Go API.

### Day 6 — Images and containers

- **Learn:** An image is a packaged, versioned filesystem; a container is a running process created from an image.
- **Practice:** Run a small public image, inspect its processes and logs, then stop and remove it.
- **Read:** [Docker: What is a container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/), [Docker: What is an image?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/).

### Day 7 — Build an image for Go

- **Learn:** Dockerfile instructions, build context, layers, and why a small, repeatable build matters.
- **Practice:** Write a Dockerfile for the API, build it, and run the resulting image.
- **Read:** [Docker: Building an image](https://docs.docker.com/get-started/docker-concepts/building-images/).

### Day 8 — Ports, environment, and persistence

- **Learn:** Container port publishing, environment-based configuration, and the difference between a container filesystem and a volume.
- **Practice:** Pass the API port through an environment variable and persist PostgreSQL data in a named volume.
- **Read:** [Docker: Persisting container data](https://docs.docker.com/get-started/docker-concepts/running-containers/persisting-container-data/).

### Day 9 — Compose the local stack

- **Learn:** Compose describes related services, their networks, configuration, and volumes in one file.
- **Practice:** Define the API and PostgreSQL in `compose.yaml`; start the stack and verify the API can reach the database.
- **Read:** [Docker Compose: Get started](https://docs.docker.com/compose/gettingstarted/).

### Day 10 — Debug and improve the image

- **Learn:** How to inspect container state, logs, network membership, and image size; introduce least-privilege runtime choices.
- **Practice:** Diagnose a deliberately incorrect database hostname, then run the API as a non-root user if the base image supports it.
- **Read:** [Docker: Containerize an application](https://docs.docker.com/get-started/docker-concepts/running-containers/).

**Milestone:** A new developer can start the API and its database with one documented Compose command.

---

## Week 3 — Git and continuous integration

**Context:** A platform should make the safe path the easy path. Begin with a reliable, automated check on code changes before adding deployment automation.

### Day 11 — Git workflow and repeatable checks

- **Learn:** Changes, commits, branches, and pull requests as a reviewable delivery path.
- **Practice:** Create a small branch and document the local commands to format, test, and build the Go API.
- **Read:** [GitHub: About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).

### Day 12 — First GitHub Actions workflow

- **Learn:** A workflow is triggered by events and runs jobs made of steps on a runner.
- **Practice:** Add a workflow triggered by pushes and pull requests; run a simple checkout and Go test job.
- **Read:** [GitHub Actions: Building and testing Go](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-go).

### Day 13 — Test and build the service

- **Learn:** CI should run the same checks contributors use locally and fail clearly when a check fails.
- **Practice:** Run formatting verification, `go test ./...`, and `go build ./...` in the workflow.
- **Read:** [GitHub Actions: Understanding workflows](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions).

### Day 14 — Cache dependencies and retain artifacts

- **Learn:** Caches speed up repeated work, while artifacts preserve outputs from a workflow run; they serve different purposes.
- **Practice:** Add Go module caching and upload a build artifact for inspection.
- **Read:** [GitHub Actions: Dependency caching](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows), [GitHub Actions: Workflow artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts).

### Day 15 — Make CI useful to contributors

- **Learn:** Clear check names, small workflows, and actionable failure messages make automation part of the developer experience.
- **Practice:** Trigger a failing test and improve the workflow so the failing step is obvious; document the local equivalent.
- **Read:** [GitHub Actions: Quickstart](https://docs.github.com/en/actions/writing-workflows/quickstart).

**Milestone:** Every pull request gets automated format, test, and build checks.

---

## Week 4 — Production-ready Go service

**Context:** Containers and orchestration need a service that behaves well during startup, failure, and shutdown. These days improve the API itself so infrastructure can operate it safely.

### Day 16 — Configuration and validation

- **Learn:** Keep environment-specific configuration outside the binary and fail early when required settings are missing or invalid.
- **Practice:** Configure the listen address and database URL through environment variables; validate them at startup.
- **Read:** [The Twelve-Factor App: Config](https://12factor.net/config), [Go: Managing dependencies](https://go.dev/doc/modules/managing-dependencies).

### Day 17 — Health and readiness

- **Learn:** A liveness check asks whether a process should be restarted; readiness asks whether it should receive traffic.
- **Practice:** Add separate health and readiness handlers and define what each checks for your API.
- **Read:** [Kubernetes: Liveness, readiness, and startup probes](https://kubernetes.io/docs/concepts/configuration/liveness-readiness-startup-probes/).

### Day 18 — Timeouts and graceful shutdown

- **Learn:** Requests and shutdowns need bounded time; graceful shutdown gives in-flight work time to finish.
- **Practice:** Configure HTTP server timeouts and handle SIGTERM by shutting down the server with a deadline.
- **Read:** [Go `net/http` package](https://pkg.go.dev/net/http), [Go `os/signal` package](https://pkg.go.dev/os/signal).

### Day 19 — Structured logs

- **Learn:** Structured logs make fields such as level, request ID, and error searchable and consistent.
- **Practice:** Emit structured startup, request, and error logs using Go's standard `log/slog` package.
- **Read:** [Go `log/slog` package](https://pkg.go.dev/log/slog).

### Day 20 — Service security basics

- **Learn:** Reduce the impact of mistakes with minimal container privileges, dependency updates, and careful secret handling.
- **Practice:** Check the image user, remove any hard-coded credentials, and add a dependency review step to the project checklist.
- **Read:** [Go: Managing dependencies](https://go.dev/doc/modules/managing-dependencies), [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html).

**Milestone:** The API validates its configuration, exposes meaningful health checks, logs structured events, and shuts down gracefully.

---

## Week 5 — Kubernetes foundations

**Context:** Kubernetes is a control system for the desired state of containerized applications. Learn the core objects by deploying the same API you already run locally.

### Day 21 — Cluster and desired state

- **Learn:** A Kubernetes cluster has a control plane and worker nodes; users declare objects and controllers work toward the declared state.
- **Practice:** Start a local cluster with kind or Minikube, inspect its nodes, and review the current context.
- **Read:** [Kubernetes: Cluster architecture](https://kubernetes.io/docs/concepts/architecture/), [Kubernetes tutorials](https://kubernetes.io/docs/tutorials/).

### Day 22 — Pods

- **Learn:** A Pod is the smallest deployable unit and groups one or more containers that share network and storage context.
- **Practice:** Create a simple Pod manifest, apply it, inspect it, read its logs, and delete it.
- **Read:** [Kubernetes: Pods](https://kubernetes.io/docs/concepts/workloads/pods/).

### Day 23 — Deployments and replica sets

- **Learn:** A Deployment describes a desired set of replicated Pods and manages updates through a ReplicaSet.
- **Practice:** Deploy the Go API with two replicas and confirm the cluster recreates a deleted Pod.
- **Read:** [Kubernetes: Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

### Day 24 — Services and in-cluster networking

- **Learn:** A Service gives a stable address to a changing set of Pods selected by labels.
- **Practice:** Expose the API inside the cluster and reach it with port-forwarding.
- **Read:** [Kubernetes: Services](https://kubernetes.io/docs/concepts/services-networking/service/).

### Day 25 — Namespaces, labels, and selectors

- **Learn:** Namespaces group objects; labels and selectors connect objects and support queries.
- **Practice:** Put the API in its own namespace, label its resources, and query them with `kubectl` selectors.
- **Read:** [Kubernetes: Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/), [Kubernetes: Labels and selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

**Milestone:** Deploy multiple API replicas in a local cluster and access them through a Kubernetes Service.

---

## Week 6 — Operating Kubernetes workloads

**Context:** Making a Pod run is only the beginning. Platform engineers need safe configuration, resource boundaries, predictable releases, and practical troubleshooting paths.

### Day 26 — ConfigMaps and Secrets

- **Learn:** ConfigMaps and Secrets provide configuration to Pods; a Secret object alone is not a complete secret-management system.
- **Practice:** Move non-sensitive settings into a ConfigMap and inject a local test credential through a Secret.
- **Read:** [Kubernetes: ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/), [Kubernetes: Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

### Day 27 — Probes and startup behavior

- **Learn:** Startup, readiness, and liveness probes solve different problems and should reflect actual service behavior.
- **Practice:** Add probes to the API Deployment and observe the result of an intentionally failing readiness check.
- **Read:** [Kubernetes: Configure liveness, readiness, and startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).

### Day 28 — CPU and memory requests and limits

- **Learn:** Requests influence scheduling; limits constrain resource use. Poor settings can cause scheduling failures or throttling.
- **Practice:** Set reasonable CPU and memory requests and limits for the local API; inspect the Pod specification.
- **Read:** [Kubernetes: Resource management for Pods and containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).

### Day 29 — Rollouts and rollbacks

- **Learn:** A Deployment rollout replaces Pods according to a strategy; history and revision commands help recover from bad changes.
- **Practice:** Update the image or configuration, watch rollout status, then roll back a deliberately broken revision.
- **Read:** [Kubernetes: Updating a Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).

### Day 30 — Troubleshooting a workload

- **Learn:** Use object status, events, logs, and container inspection in a consistent order to narrow down a failure.
- **Practice:** Diagnose a bad image name, a failed readiness check, and a Service selector mismatch.
- **Read:** [Kubernetes: Debugging applications](https://kubernetes.io/docs/tasks/debug/debug-application/).

**Milestone:** Deploy, configure, update, roll back, and troubleshoot the API without editing a running container by hand.

---

## Week 7 — Terraform and infrastructure as code

**Context:** Terraform lets teams review infrastructure changes before applying them. Learn the workflow and state model with a local lab before provisioning anything that can incur cloud charges.

### Day 31 — Configuration, providers, and initialization

- **Learn:** Terraform configuration describes resources; providers connect Terraform to APIs; initialization installs providers and modules.
- **Practice:** Create a small local Terraform project and run `terraform init` and `terraform validate`.
- **Read:** [Terraform CLI tutorials](https://developer.hashicorp.com/terraform/tutorials/cli).

### Day 32 — Plan before apply

- **Learn:** A plan previews intended changes so a human can review impact before Terraform changes managed resources.
- **Practice:** Use a local provider/resource and inspect the plan before applying it.
- **Read:** [Terraform: Create a plan](https://developer.hashicorp.com/terraform/tutorials/cli/plan).

### Day 33 — Variables and outputs

- **Learn:** Variables make configuration reusable; outputs expose selected values after apply.
- **Practice:** Parameterize a local resource with variables and export a useful output.
- **Read:** [Terraform: Input variables](https://developer.hashicorp.com/terraform/tutorials/configuration-language/variables), [Terraform: Outputs](https://developer.hashicorp.com/terraform/tutorials/configuration-language/outputs).

### Day 34 — Modules and composition

- **Learn:** Modules group related resources behind a small input/output interface and help reduce duplicated configuration.
- **Practice:** Move a local resource into a child module and call it from the root module.
- **Read:** [Terraform: Module overview](https://developer.hashicorp.com/terraform/tutorials/modules/module).

### Day 35 — State, drift, and cleanup

- **Learn:** State maps configuration to managed objects; drift occurs when real infrastructure changes outside Terraform.
- **Practice:** Inspect local state, compare a plan after a manual change, and finish with `terraform destroy`.
- **Read:** [Terraform: State](https://developer.hashicorp.com/terraform/language/state), [Terraform CLI: Apply](https://developer.hashicorp.com/terraform/tutorials/cli/apply).

**Milestone:** Explain what Terraform plans to change, where state fits, and how you safely clean up the lab.

---

## Week 8 — Reusable Kubernetes configuration

**Context:** Teams need a consistent baseline while still supporting environment-specific values. Kustomize provides a Kubernetes-native way to compose and customize manifests.

### Day 36 — Organize plain manifests

- **Learn:** Keep resource definitions declarative, reviewable, and separated by responsibility.
- **Practice:** Group the API Deployment, Service, ConfigMap, and namespace into a clean manifest directory.
- **Read:** [Kubernetes: Managing resources](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/).

### Day 37 — Kustomize bases and overlays

- **Learn:** A base defines shared resources; an overlay applies environment-specific changes without copying every manifest.
- **Practice:** Create a base and a local overlay that changes the namespace or replica count; render it before applying.
- **Read:** [Kubernetes: Declarative management with Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/).

### Day 38 — Environment differences and secrets

- **Learn:** Make differences explicit and avoid committing real credentials to version control.
- **Practice:** Add a development overlay and document how sensitive values are supplied outside Git.
- **Read:** [Kubernetes: Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/), [Kubernetes: Secrets](https://kubernetes.io/docs/concepts/configuration/secret/).

### Day 39 — Validate the deployable package

- **Learn:** Rendered manifests are the concrete interface between configuration and the cluster.
- **Practice:** Render the overlay, inspect the result, apply it to the local cluster, and verify the API.
- **Read:** [Kubernetes: Kustomize task](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/).

### Day 40 — Make deployments repeatable

- **Learn:** A repeatable deployment has documented prerequisites, a predictable command, and a verification step.
- **Practice:** Add `make deploy-local` and `make destroy-local` commands with clear output and safety checks.
- **Read:** [GNU Make manual](https://www.gnu.org/software/make/manual/make.html).

**Milestone:** Build and deploy a named local overlay using documented commands, with no manual edits to live resources.

---

## Week 9 — OpenTelemetry

**Context:** Logs describe events, metrics summarize behavior, and traces connect work across request boundaries. OpenTelemetry provides APIs and SDKs for generating and exporting telemetry.

### Day 41 — Signals and instrumentation model

- **Learn:** Understand traces, metrics, resources, attributes, and the role of an SDK/exporter.
- **Practice:** Trace one API request from handler entry to response and identify useful service-level attributes.
- **Read:** [OpenTelemetry concepts](https://opentelemetry.io/docs/concepts/), [OpenTelemetry Go](https://opentelemetry.io/docs/languages/go/).

### Day 42 — Trace HTTP requests

- **Learn:** Spans represent units of work; trace context lets related work share a trace identity.
- **Practice:** Instrument the Go HTTP server and emit traces to a local/stdout exporter.
- **Read:** [OpenTelemetry Go: Getting started](https://opentelemetry.io/docs/languages/go/getting-started/).

### Day 43 — Add useful metrics

- **Learn:** Counters track cumulative events and histograms record distributions such as request duration.
- **Practice:** Record request count and duration, separating useful dimensions such as method and status code.
- **Read:** [OpenTelemetry metrics](https://opentelemetry.io/docs/concepts/signals/metrics/), [OpenTelemetry Go](https://opentelemetry.io/docs/languages/go/).

### Day 44 — Context propagation

- **Learn:** Trace context connects work across service boundaries; context must be passed rather than recreated for every operation.
- **Practice:** Pass Go contexts through handlers and database calls; verify a request remains one trace.
- **Read:** [OpenTelemetry context propagation](https://opentelemetry.io/docs/concepts/context-propagation/).

### Day 45 — Configure shutdown and export

- **Learn:** Telemetry providers and exporters need orderly initialization and shutdown so buffered data can flush.
- **Practice:** Add telemetry setup and bounded shutdown to the service lifecycle; document the local exporter.
- **Read:** [OpenTelemetry Go: Getting started](https://opentelemetry.io/docs/languages/go/getting-started/).

**Milestone:** A request to the API produces a trace and useful request metrics with service identity attached.

---

## Week 10 — Prometheus and dashboards

**Context:** Telemetry becomes operationally useful when a team can query it, see trends, and notice when user-facing behavior degrades. Use a local monitoring stack for this course.

### Day 46 — Prometheus scrape model

- **Learn:** Prometheus collects time-series metrics by scraping configured targets.
- **Practice:** Expose a metrics endpoint and configure a local Prometheus target for the Go API.
- **Read:** [Prometheus overview](https://prometheus.io/docs/introduction/overview/), [Getting started](https://prometheus.io/docs/prometheus/latest/getting_started/).

### Day 47 — Query service metrics

- **Learn:** PromQL selects and aggregates time series; rate functions help interpret counters over time.
- **Practice:** Query request rate, error ratio, and latency distribution for the API.
- **Read:** [Prometheus: Querying basics](https://prometheus.io/docs/prometheus/latest/querying/basics/).

### Day 48 — Build a useful dashboard

- **Learn:** A dashboard should answer operational questions quickly and show trends, not just decorate metrics.
- **Practice:** Create panels for traffic, errors, latency, and resource use; add useful labels and units.
- **Read:** [Grafana: Dashboards](https://grafana.com/docs/grafana/latest/dashboards/).

### Day 49 — Alert on symptoms

- **Learn:** Alerts should identify user-impacting symptoms and tell the responder what to inspect next.
- **Practice:** Create one local alert rule for sustained API errors or unavailable targets and verify its firing state.
- **Read:** [Prometheus: Alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/).

### Day 50 — Exercise the signal path

- **Learn:** Monitoring must be tested end to end: generate a failure, observe telemetry, and confirm the signal is understandable.
- **Practice:** Stop the API or make it return errors; follow the change from metric to dashboard/alert and write down the diagnosis.
- **Read:** [Prometheus: Alerting](https://prometheus.io/docs/alerting/latest/overview/).

**Milestone:** Detect a local service failure from metrics and explain the likely user impact using a dashboard.

---

## Week 11 — Platform as a product

**Context:** A platform serves developers as its users. Start with a repeated developer task, provide a paved path with safe defaults, and make its interface discoverable and documented.

### Day 51 — Identify the developer journey

- **Learn:** A platform capability should solve a real recurring friction point and have clear users and outcomes.
- **Practice:** Write the steps a developer currently takes to start, test, and deploy the sample Go service; mark avoidable manual steps.
- **Read:** [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/).

### Day 52 — Define a golden path

- **Learn:** A golden path is a supported, opinionated route that makes common work easier while keeping escape hatches possible.
- **Practice:** Define the minimum supported path for a new Go service: scaffold, CI, local run, and local deployment.
- **Read:** [Backstage Software Templates](https://backstage.io/docs/features/software-templates/).

### Day 53 — Create a service template

- **Learn:** Templates encode good defaults and reduce repetitive setup without hiding how the service works.
- **Practice:** Turn the sample service structure into a reusable template or documented copyable scaffold.
- **Read:** [Backstage Software Templates](https://backstage.io/docs/features/software-templates/).

### Day 54 — Add self-service commands

- **Learn:** A platform interface should be consistent, discoverable, and provide useful feedback when steps fail.
- **Practice:** Add `make test`, `make image`, `make deploy-local`, and `make status` commands with prerequisites and error messages.
- **Read:** [GNU Make manual](https://www.gnu.org/software/make/manual/make.html).

### Day 55 — Document the user experience

- **Learn:** Documentation is part of the platform interface; a new user should be able to reach a working service without tribal knowledge.
- **Practice:** Ask someone unfamiliar with the repo (or follow the README from a clean checkout) to use the golden path; fix confusing steps.
- **Read:** [Backstage: Software Catalog](https://backstage.io/docs/features/software-catalog/).

**Milestone:** A developer can start from the project template and reach a healthy local deployment using documented commands.

---

## Week 12 — Reliability and capstone

**Context:** Platform engineering and SRE overlap in automation and observability. This final week adds reliability thinking: define what users need, choose signals, respond to failures, and demonstrate the complete developer path.

### Day 56 — SLIs, SLOs, and error budgets

- **Learn:** An SLI measures a user-relevant aspect of service behavior; an SLO sets a target over a time window; an error budget describes allowed unreliability.
- **Practice:** Define a request-availability SLI and a draft SLO for the sample API. State what user experience the metric represents.
- **Read:** [Google SRE: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/), [The SRE Workbook](https://sre.google/workbook/).

### Day 57 — Runbooks and response

- **Learn:** A runbook turns an alert into safe first steps, useful diagnostic links, and a clear escalation path.
- **Practice:** Write a runbook for high API errors and one for an unavailable database dependency.
- **Read:** [Google SRE: Being On-Call](https://sre.google/sre-book/being-on-call/).

### Day 58 — Failure and recovery exercise

- **Learn:** A blameless incident review focuses on impact, timeline, contributing conditions, and system improvements.
- **Practice:** Simulate a dependency failure, follow your dashboard and runbook, restore service, and write a short incident review.
- **Read:** [Google SRE: Postmortem Culture](https://sre.google/sre-book/postmortem-culture/).

### Day 59 — Capstone integration and review

- **Learn:** A platform capability is a complete path across source, checks, packaging, deployment, and operations.
- **Practice:** From a clean checkout, follow your golden path; verify CI, container build, Kubernetes deployment, telemetry, and cleanup.
- **Read:** Review the relevant official docs from [Kubernetes](https://kubernetes.io/docs/home/), [Terraform](https://developer.hashicorp.com/terraform/docs), and [OpenTelemetry](https://opentelemetry.io/docs/).

### Day 60 — Demo and next steps

- **Learn:** A strong portfolio explains the user problem, system design, operational choices, trade-offs, and what you would improve next.
- **Practice:** Record a short written walkthrough with architecture, setup commands, dashboard screenshots, one failure scenario, and lessons learned. No video is required.
- **Read:** Review your project README and [CNCF Platforms White Paper](https://tag-app-delivery.cncf.io/whitepapers/platforms/).

**Final outcome:** A documented Go service platform demo with CI, container packaging, local Kubernetes deployment, Terraform practice, observability, a developer golden path, and a basic reliability runbook.

## Completion checklist

- [ ] Each week's lab has a README with prerequisites, commands, expected result, and cleanup instructions.
- [ ] The Go service can be built and tested from a clean checkout.
- [ ] CI reports test and build status on pull requests.
- [ ] Docker Compose starts the local development stack.
- [ ] Kubernetes manifests can be rendered, applied, verified, and removed locally.
- [ ] Terraform changes are reviewed with a plan and the lab is destroyed afterward.
- [ ] A request can be followed through logs, metrics, and traces.
- [ ] The golden path and runbook work for someone other than their author.
