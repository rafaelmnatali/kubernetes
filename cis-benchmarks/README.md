# Running CIS Benchmarks in a K8s cluster

- [Running CIS Benchmarks in a K8s cluster](#running-cis-benchmarks-in-a-k8s-cluster)
  - [Summary](#summary)
  - [Installing kube-bech](#installing-kube-bech)
  - [Running kube-bech](#running-kube-bech)

## Summary

This tutorial demonstrates how to execute a [CIS Benchmark for Kubernetes](https://www.cisecurity.org/benchmark/kubernetes) using [kube-bench](https://github.com/aquasecurity/kube-bench) to check whether Kubernetes is deployed securely accordingly to industry standards.

## Installing kube-bech

See the [official documentation](https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md) and chose your preferred installation method.

This tutorial will use the [Quick Start method](https://github.com/aquasecurity/kube-bench?tab=readme-ov-file#quick-start).

## Running kube-bech

See the [official documentation](https://github.com/aquasecurity/kube-bench/blob/main/docs/running.md) nd chose your preferred running method.

This tutorial will use the [Running in a Kubernetes cluster option](https://github.com/aquasecurity/kube-bench/blob/main/docs/running.md#running-in-a-kubernetes-cluster) with the following [kube-bench-job.yaml](../manifests/kube-bench-job.yaml).
