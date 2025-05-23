---
title: "Developer Guide"
description: "This section tells how to develop Envoy Gateway."
weight: 2
---

Envoy Gateway is built using a [make][]-based build system. Our CI is based on [Github Actions][] using [workflows][].

## Prerequisites

### go

* Version: 1.24
* Installation Guide: https://go.dev/doc/install

### make

* Recommended Version: 4.0 or later
* Installation Guide: https://www.gnu.org/software/make

### docker

* Optional when you want to build a Docker image or run `make` inside Docker.
* Recommended Version: 20.10.16
* Installation Guide: https://docs.docker.com/engine/install

### python3

* Need a `python3` program
* Must have a functioning `venv` module; this is part of the standard
  library, but some distributions (such as Debian and Ubuntu) replace
  it with a stub and require you to install a `python3-venv` package
  separately.

## Quickstart

Run `make help` to see all the available targets to build, test and run Envoy Gateway. Below are the other make directives

### Building

* Run `make build` to build all the binaries.
* Run `make build BINS="envoy-gateway"` to build the Envoy Gateway binary.
* Run `make build BINS="egctl"` to build the egctl binary.

__Note:__ The binaries get generated in the `bin/$OS/$ARCH` directory, for example, `bin/linux/amd64/`.

### Testing

* Run `make test` to run the golang tests.

* Run `make e2e` to perform end-to-end testing.

* Run `make testdata` to generate the golden YAML testdata files.

### Running Linters

* Run `make lint` to make sure your code passes all the linter checks.

__Note:__ The `golangci-lint` configuration resides [here](https://github.com/envoyproxy/gateway/blob/main/tools/linter/golangci-lint/.golangci.yml).

### Building and Pushing the Image

* Run `IMAGE=docker.io/you/gateway-dev make image` to build the docker image.
* Run `IMAGE=docker.io/you/gateway-dev make push-multiarch` to build and push the multi-arch docker image.

__Note:__  Replace `IMAGE` with your registry's image name.

### Raising a PR

* Run `make generate` and push the generated files along with your commit, if your PR contains any **API** changes (changes in `/api` folder), you've added some unit tests or you've updated the modules used in the project. 

### Deploying Envoy Gateway for Test/Dev

* Run `make create-cluster` to create a [Kind][] cluster.

#### Option 1: Use the Latest [gateway-dev][] Image

* Run `TAG=latest make kube-deploy` to deploy Envoy Gateway in the Kind cluster using the latest image. Replace `latest`
  to use a different image tag.

#### Option 2: Use a Custom Image

* Run `make kube-install-image` to build an image from the tip of your current branch and load it in the Kind cluster.
* Run `IMAGE_PULL_POLICY=IfNotPresent make kube-deploy` to install Envoy Gateway into the Kind cluster using your custom image.

### Deploying Envoy Gateway in Kubernetes

* Run `TAG=latest make kube-deploy` to deploy Envoy Gateway using the latest image into a Kubernetes cluster (linked to
  the current kube context). Preface the command with `IMAGE` or replace `TAG` to use a different Envoy Gateway image or
  tag.
* Run `make kube-undeploy` to uninstall Envoy Gateway from the cluster.

__Note:__ Envoy Gateway is tested against Kubernetes v1.24.0.

### Demo Setup

* Run `make kube-demo` to deploy a demo backend service, gatewayclass, gateway and httproute resource
(similar to steps outlined in the [Quickstart][] docs) and test the configuration.
* Run `make kube-demo-undeploy` to delete the resources created by the `make kube-demo` command.

### Run Gateway API Conformance Tests

The commands below deploy Envoy Gateway to a Kubernetes cluster and run the Gateway API conformance tests. Refer to the
Gateway API [conformance homepage][] to learn more about the tests. If Envoy Gateway is already installed, run
`TAG=latest make run-conformance` to run the conformance tests.

#### On a Linux Host

* Run `TAG=latest make conformance` to create a Kind cluster, install Envoy Gateway using the latest [gateway-dev][]
  image, and run Gateway API conformance tests.

#### On a Mac Host

Since Mac doesn't support [directly exposing][] the Docker network to the Mac host, use one of the following
workarounds to run conformance tests:

* Deploy your own Kubernetes cluster or use Docker Desktop with [Kubernetes support][] and then run
  `TAG=latest make kube-deploy run-conformance`. This will install Envoy Gateway using the latest [gateway-dev][] image
  to the Kubernetes cluster using the current kubectl context and run the conformance tests. Use `make kube-undeploy` to
  uninstall Envoy Gateway.
* Install and run [Docker Mac Net Connect][mac_connect] and then run `TAG=latest make conformance`.

__Note:__  Preface commands with `IMAGE` or replace `TAG` to use a different Envoy Gateway image or tag. If `TAG`
is unspecified, the short SHA of your current branch is used.

### JWT Testing

An example [JSON Web Token (JWT)][jwt] and [JSON Web Key Set (JWKS)][jwks] are used for the [request authentication][]
task. The JWT was created by the [JWT Debugger][], using the `RS256` algorithm. The public key from the JWTs
verify signature was copied to [JWK Creator][] for generating the JWK. The JWK Creator was configured with matching
settings, i.e. `Signing` public key use and the `RS256` algorithm. The generated JWK was wrapped in a JWKS structure
and is hosted in the repo.

### Benchmarking

Envoy Gateway uses [nighthawk][nighthawk] for benchmarking, and mainly concerned with
its performance and scalability as a control-plane.

The performance and scalability concerns come from several aspects for control-plane:

- The consumption of memory and CPU.
- The rate of configuration changes.

The benchmark test is running on a [Kind][Kind] cluster, you can start a Kind cluster and 
run benchmark test on it by executing `make benchmark`.

The benchmark report will be included in the release artifacts, you can learn more by downloading
the detailed benchmark report, namely `benchmark_report.zip`.

Here are some brief benchmark reports about Envoy Gateway: 

- It will take up nearly 550MiB memory and 11s total CPU time for (1 GatewayClass + 1 Gateway + 500 HTTRoutes) settings


[Quickstart]: https://github.com/envoyproxy/gateway/blob/main/docs/latest/user/quickstart.md
[make]: https://www.gnu.org/software/make/
[Github Actions]: https://docs.github.com/en/actions
[workflows]: https://github.com/envoyproxy/gateway/tree/main/.github/workflows
[Kind]: https://kind.sigs.k8s.io/
[conformance homepage]: https://gateway-api.sigs.k8s.io/concepts/conformance/
[directly exposing]: https://kind.sigs.k8s.io/docs/user/loadbalancer/
[Kubernetes support]: https://docs.docker.com/desktop/kubernetes/
[gateway-dev]: https://hub.docker.com/r/envoyproxy/gateway-dev/tags
[mac_connect]: https://github.com/chipmk/docker-mac-net-connect
[jwt]: https://tools.ietf.org/html/rfc7519
[jwks]: https://tools.ietf.org/html/rfc7517
[request authentication]: ../latest/tasks/security/jwt-authentication
[JWT Debugger]: https://jwt.io/
[JWK Creator]: https://russelldavies.github.io/jwk-creator/
[nighthawk]: https://github.com/envoyproxy/nighthawk
