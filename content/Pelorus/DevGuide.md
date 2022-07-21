---
title: "Pelorus Development Guide"
date: 2022-07-14T15:16:11-06:00
draft: false
---

Contributing to Pelorus is appreciated and encouraged. Use this guide to get up and running. There are three main Pelorus development tracks to consider.

**Deployment Automation**
This track mostly involves testing, fixing, and updating our Helm chart(s) to streamline the installation and configuration experience. Experience with configuring Helm, OpenShift, Operators, and Prometheus is required for this work.

**Dashboard Development**
This is where we take the collected raw data and turn it into actionable, visual representations that will help IT organizations make important decisions. Knowledge of Grafana and PromQL is required for contribution.

**Exporter Development**
This track is focused around the development of custom [Prometheus exporters](https://prometheus.io/docs/instrumenting/writing_exporters/) to gather information to calculate our core metrics. Python development experience is required.

## Contributing to Deployment Automation
[Helm](https://helm.sh/) is used to provide an automated deployment and configuration experience for Pelorus. We are always working to cover more and more complex use cases with our Helm charts. In order to be able to effectively contribute to these charts, you will need a cluster that satisfies all of the installation prerequisites for Pelorus.

See the [Installation section](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/installation.md) for more details.

Currently there are two charts:

* The [Operators chart](https://github.com/konveyor/pelorus/tree/master/charts/operators) installs the community operators that Pelorus depends on.
    * [Prometheus Operator](https://operatorhub.io/operator/prometheus)
    * [Grafana Operator](https://operatorhub.io/operator/grafana-operator)

* The [Pelorus chart](https://github.com/konveyor/pelorus/tree/master/charts/pelorus) manages the Pelorus stack including:
    * Prometheus
    * Thanos
    * Grafana
    * A set of Grafana Dashboards and Datasources
    * The Pelorus exporters, managed in an [exporter](https://github.com/konveyor/pelorus/tree/master/charts/pelorus/charts/exporters) subchart.

Pelorus uses Helm's [chart-testing tool](https://github.com/helm/chart-testing) to ensure quality and consistency in the chart. When making updates to one of the charts, ensure that the chart still passes lint testing using `make chart-lint`. The most common linting failure is forgetting to bump the `version` field in the `Chart.yaml`. See below for instructions on updating the version.

## Updating the chart versions
When any Helm charts are updated, we need to bump the version number for a seemless upgrade experience. We have provided scripts that can test when a version bump is needed and do the bumping for you.

**Procedure**

1. Verify the development environment is set up with make dev-env.
2. Run make chart-lint to lint the charts, including checking the version number.

> **Note:** (Optional) Check all chart versions and bump them if needed with a script that compares upstream Pelorus repository with the changes in a fork.

4. Verify the upstream repository is added to the fork.
```
$ git remote add upstream https://github.com/konveyor/pelorus.git
$ git pull
$ make chart-check-bump
```
or bump specific charts with shell script:
```
$ ./scripts/bump-version CHART_PATH [ CHART_PATH ...]
```

## Dashboard Development
Pelorus is continually working to enhance and bugfix the dashboards. Doing so requires a complete Pelorus stack, including all exporters required to populate a given dashboard. See the [Dashboards section](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/dashboard.md) for more information.

For effective dashboard development, you will likely need at least two browser windows open, one with Grafana, and another with Prometheus for testing queries. Since our dashboards are imported to Grafana via the Grafana Operator, they get imported in read-only mode. Because of this, you will need to make a copy of it for development purposes.

The following steps outline a workflow for working on a dashboard:

**Procedure**

1. Get the Grafana route to sign in.
```
$ oc get route grafana-route -n pelorus
```
2. Sign in as an administrator.
3. Click the **Sign In** button in the bottom right corner.
4. Pull the admin credentials.
```
$ oc get secrets -n pelorus grafana-admin-credentials -o jsonpath='{.data.GF_SECURITY_ADMIN_USER}' | base64 -d
$ oc get secrets -n pelorus grafana-admin-credentials -o jsonpath='{.data.GF_SECURITY_ADMIN_PASSWORD}' | base64 -d
```
5. Export the dashboard JSON.
    a. Open the dashboard, and select the **Share...** button.
    b. Select the **Export** tab.
    c. Click **View JSON**.
    d. Click **Copy to Clipboard**.
4. Import as a new dashboard
    a. Click **Create**, then **Import**.
    b. Paste the JSON code in the box and click **Load**.
    c. Change the **Name** and **Unique Identifier** fields and click **Import**.
5. Click the drop-down list by the panel names and click **Edit** to make changes to the live dashboard.
6. Verify the changes, then export the updated dashboard and replace the existing content in the GrafanaDashbaord CR.
    a. Open the dashboard and click the **Share...** button.
    b. Click the **Export** tab.
    c. Click **View JSON**.
    d. Click **Copy to Clipboard**.
    e. Open the appropriate `GrafanaDashboard` CR file, and paste the new dashboard JSON over the existing.

> **Important:** Match the indentation of the previous dashboard JSON. The git diffs should still show only the lines changed like in the example below.
```
 $ git diff charts/deploy/templates/metrics-dashboard.yaml
 diff --git a/charts/deploy/templates/metrics-dashboard.yaml b/charts/deploy/templates/metrics-dashboard.yaml
 index 73151ad..c470afc 100644
 --- a/charts/deploy/templates/metrics-dashboard.yaml
 +++ b/charts/deploy/templates/metrics-dashboard.yaml
 @@ -25,7 +25,7 @@ spec:
             "editable": true,
             "gnetId": null,
             "graphTooltip": 0,
 -            "id": 2,
 +            "id": 3,
             "links": [],
             "panels": [
                 {
 @@ -323,7 +323,7 @@ spec:
                 "tableColumn": "",
                 "targets": [
                     {
 -                    "expr": "count (deploy_timestamp)",
 +                    "expr": "count (count_over_time (deploy_timestamp [$__range]) )",
                     "format": "time_series",
                     "instant": true,
                     "intervalFactor": 1,
 @@ -410,7 +410,7 @@ spec:
````

7. Commit your changes and open a PR.

## Exporter Development
A Pelorus exporter is simply a [Prometheus exporter](https://prometheus.io/docs/instrumenting/writing_exporters/). While they can technically be written in many languages, Pelorus' are written in Python using the [Prometheus Python client library](https://github.com/prometheus/client_python). We chose Python because it seems to be the most popular programming language for operations teams.

### Exporter directory layout
The following is a recommended directory structure for a Pelorus exporter `<NAME>`.
```
.
├── charts
│   └── pelorus
│       ├── configmaps
│       │   └── <NAME>.yaml
│       └── values.yaml
└── exporters
    ├── <NAME>
    │   ├── app.py
    │   └── README.md
    └── tests
        └── test_<NAME>.py
```
## Python Dev Environment Setup and and Repo Setup
Install Python (version >= 3.9 but < 3.11) after cloning the repo. Running `make dev-env` should be enough to get started.

This will:
* check for the right version of python
* set up a virtual environment
* install required CLI tools such as helm, oc, tkn and ct, promtool, conftest (inside .venv/bin)
* install required python runtime and test dependencies
* install the exporters package
* set up git hooks for formatting and linting checks
* configure git blame to ignore large revisions that just changed formatting

### IDE Setup (VSCode)
Most developers use Visual Studio Code for Python development. The following extensions for VSCode are useful. Each can be installed by hitting `Ctrl+P` and pasting the commands below.

**Procedure**

1. Install [Markdown Preview Github Styling](https://marketplace.visualstudio.com/items?itemName=bierner.markdown-preview-github-styles)
```
ext install bierner.markdown-preview-github-styles
```
2. Install [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
```
ext install ms-python.python
```
> **Note:** The Python extension can activate the virtualenv automatically.

3. Create a file called `.vscode/launch.json` in your `pelorus/` project directory with the following content for a starter VSCode degug configuration to use with Pelorus exporters.
```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Commit Time Exporter",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/exporters/committime/app.py",
            "console": "integratedTerminal",
            "env": {
                "GITHUB_USER": "<github username here>",
                "GITHUB_TOKEN": "<personal access token here>",
                "LOG_LEVEL": "INFO",
                "APP_LABEL": "app.kubernetes.io/name"
            }
        },
        {
            "name": "Deploy Time Exporter",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/exporters/deploytime/app.py",
            "console": "integratedTerminal",
            "env": {
                "LOG_LEVEL": "INFO",
                "APP_LABEL": "app.kubernetes.io/name"
            }        },
        {
            "name": "Deploy Time Exporter",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/exporters/failure/app.py",
            "console": "integratedTerminal",
            "env": {
                "SERVER": "<Jira server url>",
                "PROJECT": "<Jira project ID>",
                "USER": "<Jira username>",
                "TOKEN": "<Jira personal access token>",
                "LOG_LEVEL": "INFO",
                "APP_LABEL": "app.kubernetes.io/name"
            }
        }
    ]
}
```
For more information, see the [Debugging](https://code.visualstudio.com/docs/editor/debugging) doc in VSCode.

## Running exporters locally
Follow the steps below to run an exporter on a local machine.

**Procedure**

1. Set up your local dev environment.
```
make dev-env
```
2. Activate your virtual environment.
```
. .venv/bin/activate
```
3. Set any environment variables required (or desired) for the given exporter ([see Configuration exporters](https://konveyor.github.io/pelorus/configuration/#configuring-exporters) for supported variables).
```
export LOG_LEVEL=debug
export TOKEN=xxxx
export USER=xxxx
```
4. Log into the OpenShift cluster OR export the KUBECONFIG environment variable.
```
oc login --token=<token> --server=https://api.cluster-my.fun.domain.com:6443

# OR

export KUBECONFIG=/path/to/kubeconfig_file
```
5. (Optional) Avoid certificate warnings and some possible errors by setting up your local machine to trust your cluster certificate
    * a. Download your cluster ca.crt file.
    * b. Add cert to the system trust bundle.
    * c. Pass the cert bundle with your login command.
```
oc login --token=<token> --server=https://api.cluster-my.fun.domain.com:6443  --certificate-authority=/etc/pki/tls/certs/ca-bundle.crt
```
6. Start the exporter.
```
python exporters/committime/app.py
```
7. Find the exporter at http://localhost:8080.
```
curl http://localhost:8080
```

### Testing pull requests
Follow the steps below for testing Pull Requests for specific types of changes.

**Procedure:**

1. Checkout the PR. (Recommend using [GitHub CLI](https://cli.github.com/), which simplifies process of pulling PRs.)
2. Verify you have [Pelorus GitHub](https://github.com/konveyor/pelorus) project Forked into your GitHub user space.
3. Checkout the PR on top of the fork.
```
git clone git@github.com:<your_github_username>/pelorus.git
cd pelorus
gh pr checkout 535

# If asked:
# ? Which should be the base repository, select:
# > konveyor/pelorus
```

### Making Dashboard Changes
Follow the steps below to make changes to the dashboard.

> **Note:**: In most cases you can deploy changes to an existing deployment to retain existing data.

**Procedure**
1. [Checkout](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/DevGuide.md) the PR on top of your fork.
2. [Install Pelorus](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/installation.md) from checked out fork/branch.
3. Log into Grafana via the grafana route.
```
oc get route grafana-route -n pelorus
```
4. Click on the dashboard containing changes, and visually validate the behavior change described in the PR.

> **Note:** Eventually Pelorus would like to have some Selenium tests in place to validate dashboards. If you have skills in this area let us know.

### Exporter Changes
Each PR runs exporter tests in the CI systems, however those changes can be tested locally in a very similar way they run in the CLI. Follow the steps below to export the changes.

1. [Checkout](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/DevGuide.md) the PR on top of your fork.

2. Set up the dev environment.
```
make dev-env
```
3. Activate your virtual environment.
```
. .venv/bin/activate
```
4. Check what type of tests you can run.
```
make help
```
5. As an example run unit tests using `make unit-tests`.

> **Note:** You can also run coverage reports with the following:
```
coverage run -m pytest -rap -m "not integration and not mockoon"
coverage report
```

6. Gather necessary [configuration information](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/configuration.md).

7. [Run exporter locally](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/DevGuide.md) using the command line the provided VSCode [debug configuration](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/DevGuide.md) to run it in your IDE Debugger.
8. Once exporter is running, test it via `curl localhost:8080` and validate:
    * A valid response with metrics.
    * The format of expected metrics.

### Making Helm install changes
For testing changes to the Helm chart, follow the [standard install process](https://pelorus.readthedocs.io/en/latest/installation.md), then verify that:

* All expected pods are running and healthy.
* Any expected behavior changes mentioned in the PR can be observed.

Another way to test changes is by running e2e-tests against your cluster. To do so, verify the Pelorus namespace either does not exist or no resources are within that namespace.
```
​export KUBECONFIG=/path/to/kubeconfig_file
# DANGEROUS as this will remove your previously deployed pelorus instance
oc delete namespace pelorus
make e2e-tests
# Check if command ran without failures
echo $?
```
> **Note:** Rudimentary linting with `make chart-lint` can be done.

Pelorus is in the process of refactoring our Helm charts so that they can be tested more automatically using [helm chart-testing](https://github.com/helm/chart-testing). Some general guidelines are outlined in the [CoP Helm testing strategy](https://redhat-cop.github.io/ci/linting-testing-helm-charts.html).

## Release Management Process
The following steps are a walkthrough of the process to create and manage versioned releases of Pelorus. Pelorus release versions follow SemVer versioning conventions. Change of the version is managed via Makefile.

**Procedure**

1. Create Pelorus Pull Request with the release you are about to make.

    * For PATCH version bump use:
```
make release
```
    * For minor-release version:
```
make minor-release
```
    * For major-release version:
```
make major-release
```
2. Propose Pull Request to the project github repository.
> **Important:**Verify the PR is labeled with "minor" or "major" if one was created.

3. After PR is merged on the [Pelorus releases page](https://github.com/konveyor/pelorus/releases), click **Edit** on the latest Draft.
4. Click **Publish Release**.

[Source](https://github.com/konveyor/konveyor.github.io/blob/main/content/Pelorus/DevGuide.md)