[![License](https://img.shields.io/hexpm/l/plug.svg?maxAge=2592000)]()
![Generate README.html](https://github.com/garethahealy/pluto-blog/workflows/Generate%20README.html/badge.svg)

# Detecting deprecated APIs with Pluto in your CI/CD workflows
As the Kubernetes ecosystem evolves so have the APIs provided which has lead to [deprecations](https://kubernetes.io/docs/reference/using-api/deprecation-guide/) and removals.
If you have moved to Kubernetes 1.22 or OpenShift 4.9 you might have already felt some of that pain.

For those of whom have not upgraded or are thinking of upgrading in the coming months, 
[Pluto](https://github.com/FairwindsOps/pluto) which is an open-source project,
that is not sponsored by Red Hat nor is it supported under a Red Hat subscription, is here to help:

> Pluto is a utility to help users find deprecated Kubernetes apiVersions in their code repositories and their helm releases.

In simple terms; it is a command-line tool that scans the Group Version Kind (GVK) of resources and validates against a version file, such as the below:

```yaml
deprecated-versions:
- version: extensions/v1beta1
  kind: Deployment
  deprecated-in: v1.9.0
  removed-in: v1.16.0
  replacement-api: apps/v1
  component: k8s
target-versions:
  k8s: v1.22.0
```

## OK, but why do I need pluto?
If you've been using k8s/OCP for a while, it should be easy to imagine the following:

> You have multiple teams with multiple applications on the platform. Some of those applications
> use an auxiliary service, such as MongoDB, which is provided by another team or external partner via a helm chart.
> A web of YAML dependencies has inadvertently been created. Do you know what `apiVersion` the `Deployment` of MongoDB uses?

Of course not, your concern should be the business value created by your application.
But that MongoDB service is within the remit of your team to run which means it is down to your team to fix and mitigate any issues.

This is where `pluto` helps. Once it is integrated into your CI/CD pipeline, you can scan your resources against the latest k8s version via:

```bash
helm template {chart dir} | pluto detect -
```

Or, by specifying a target version for the component, if you want to pin against a specific version or look-ahead:

```bash
$ helm template charts/v1 | pluto detect --target-versions k8s=v1.20.0 -
```

## Cool, so how do I check my OCP resources?
As Pluto uses a simple `version.yaml` to determine what resource should be triggered against, it is possible to write a versions
file, like the below, which would work for `DeploymentConfigs`. The strength of Pluto is that it is not tied to any single stack or component,
if your YAML file contains a GVK, you can use Pluto against it.

```yaml
deprecated-versions:
- version: v1
  kind: DeploymentConfig
  deprecated-in: v3.11.0
  removed-in: v4.9.0
  replacement-api: apps.openshift.io/v1
  component: ocp
target-versions:
  ocp: v4.9.0
```

```bash
$ helm template charts/v1 | pluto detect --additional-versions ocp-versions.yaml -
```

## So how do I include this in my workflow?
As it's a binary, you just need an extra stage/step that executes the binary. If you are using `Jenkins`:

```groovy
stage("Run pluto against chart") {
    sh "helm template chart/ | pluto detect -"
}
```

If you are using [GitHub workflows](https://github.com/garethahealy/pluto-blog/actions/workflows/tests.yaml):

```yaml
- name: Download Pluto
  uses: FairwindsOps/pluto/github-action@master

- name: Run pluto against chart
  run: |
    helm template chart/ | pluto detect -
```

## What is next?
The next step for me is to contribute a `tekton` [task](https://github.com/FairwindsOps/pluto/issues/226) when I have some downtime.

I am interested in exploring the possibility of programmatically creating a `versions.yaml` file based on the APIServer output. If anyone
has any suggestions, I am all ears.