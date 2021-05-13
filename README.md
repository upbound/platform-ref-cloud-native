# Cloud Native Reference Platform

This reference platform `Configuration` for Cloud Native applications and platform services is a
starting point to build, run, and operate your own internal cloud platform and offer a self-service
console and API to your internal teams.

It provides platform APIs to provision fully configured workload clusters, with secure networking
and a complete set of platform services that provide the foundation for running cloud native
applications.  This is a good example of how a platform team may provide a complete platform for
cloud native applications to run on, which includes continuous deployment, monitoring, metrics,
logging, etc.

The application team simply has to request a "cloud native platform" cluster through this
declarative platform API and they'll be ready to deploy their workloads to a complete platform in no
time.

## Quick Start

### Platform Ops/SRE: Run your own internal cloud platform

#### Create a free account in Upbound Cloud

1. Sign up for [Upbound Cloud](https://cloud.upbound.io/register).
1. Create an `Organization` for your teams.

#### Create a Platform instance in Upbound Cloud

1. Create a `Platform` in Upbound Cloud (e.g. dev, staging, or prod).
1. Connect `kubectl` to your `Platform` instance.

#### Install the Crossplane kubectl extension (for convenience)

```console
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
cp kubectl-crossplane /usr/local/bin
```

#### Install the Platform Configuration

```console
PLATFORM_CONFIG=registry.upbound.io/upbound/platform-ref-cloud-native:v0.0.2

kubectl crossplane install configuration ${PLATFORM_CONFIG}
kubectl get pkg
```

#### GCP Provider Setup

Set up your GCP account keyfile by following the instructions on:
https://crossplane.io/docs/v1.0/getting-started/install-configure.html#select-provider

Ensure that the following roles are added to your service account:

* `roles/compute.networkAdmin`
* `roles/container.admin`
* `roles/iam.serviceAccountUser`

Then create the secret using the given `creds.json` file:

```console
kubectl create secret generic gcp-creds -n crossplane-system --from-file=key=./creds.json
```

Create the `ProviderConfig`, ensuring to set the `projectID` to your specific GCP project:

```console
kubectl apply -f examples/provider-default-gcp.yaml
```

### App Dev/Ops: Consume the infrastructure you need using kubectl

#### Provision a Cloud Native platform cluster in your team Workspace GUI console

1. Browse the available self-service APIs (XRDs) in your team `Workspace`
1. Provision a `Cluster` using the custom generated GUI for your
Platform `Configuration`
1. View status / details in your `Workspace` GUI console

### Cleanup & Uninstall

#### Cleanup Resources

There are 2 options to delete resources created through the `Workspace` GUI:

* From the `Workspace` GUI using the ellipsis menu in the resource view.
* Using `kubectl delete -n team1 <claim-name>`.

Verify all underlying resources have been cleanly deleted:

```console
kubectl get managed
```

#### Uninstall Provider & Platform Configuration

```console
kubectl delete configurations.pkg.crossplane.io platform-ref-cloud-native
kubectl delete providers.pkg.crossplane.io provider-gcp
kubectl delete providers.pkg.crossplane.io provider-helm
```

## APIs in this Configuration

* `Cluster` - provision a fully configured Kubernetes cluster
  * [definition.yaml](cluster/definition.yaml)
  * [composition.yaml](cluster/composition.yaml) includes (transitively):
    * `GKECluster`
    * `NodePool`
    * `Network`
    * `Subnetwork`
    * `HelmReleases` for Prometheus, Jaeger, Fluentd, Rook, and Flux platform services.
* `Network` - fabric for a `Cluster` to securely connect the control plane, pods, and services
  * [definition.yaml](network/definition.yaml)
  * [composition.yaml](network/composition.yaml) includes:

## Customize for your Organization

Create a `Repository` called `platform-ref-cloud-native` in your Upbound Cloud `Organization`.

Set these to match your settings:

```console
UPBOUND_ORG=acme
UPBOUND_ACCOUNT_EMAIL=me@acme.io
REPO=platform-ref-cloud-native
VERSION_TAG=v0.0.2
REGISTRY=registry.upbound.io
PLATFORM_CONFIG=${REGISTRY:+$REGISTRY/}${UPBOUND_ORG}/${REPO}:${VERSION_TAG}
```

Clone the GitHub repo.

```console
git clone https://github.com/upbound/platform-ref-cloud-native.git
cd platform-ref-cloud-native
```

Login to your container registry.

```console
docker login ${REGISTRY} -u ${UPBOUND_ACCOUNT_EMAIL}
```

Build package.

```console
kubectl crossplane build configuration --name package.xpkg --ignore "examples/*,hack/*"
```

Push package to registry.

```console
kubectl crossplane push configuration ${PLATFORM_CONFIG} -f package.xpkg
```

Install package into an Upbound `Platform` instance.

```console
kubectl crossplane install configuration ${PLATFORM_CONFIG}
```

To learn more see [Configuration
Packages](https://crossplane.io/docs/v0.14/getting-started/package-infrastructure.html).

## Learn More

If you're interested in building your own reference platform for your company,
we'd love to hear from you and chat. You can setup some time with us at
info@upbound.io.

For Crossplane questions, drop by [slack.crossplane.io](https://slack.crossplane.io), and say hi!
