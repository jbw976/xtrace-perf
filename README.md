# Trace Perf Testing

This repo tests the performance of the [`crossplane beta
trace`](https://docs.crossplane.io/latest/cli/command-reference/#beta-trace)
command. When a `XTracePerf` resource is created, its underlying composition
will create a variable number of resources. When large numbers are used, such as
100+, the performance of the `crossplane beta trace` command can be tested, as
it will need to make many API calls to discover the full tree of composed
resources belonging to the parent `XTracePerf` composite resource.

## Pre-reqs

* A Kubernetes cluster has been created and you have `kubectl` access
* Crossplane has been installed on the cluster ([installation
  docs](https://docs.crossplane.io/latest/software/install/))

## Setup

In a live cluster with Crossplane installed, run the following to install the
necessary functions and providers:
```
kubectl apply -f functions.yaml
kubectl apply -f provider.yaml
```

Wait for all packages to be installed and become healthy:
```
kubectl get pkg
```

Apply the `provider-kubernetes` `ProviderConfig` that gives it access to the
cluster:
```
kubectl apply -f providerconfig.yaml
```

Apply the `XRD` and `Composition` to the control plane:
```
kubectl apply -f definition.yaml
kubectl apply -f composition.yaml
```

Now create the trace perf tester object, setting its `resourceCount` value to
the number of resources you want to create:
```
kubectl apply -f claim.yaml
```

Now you can run the `crossplane beta trace` command to see how long it takes to
discover all resources in the tree:
```
❯ time crossplane beta trace traceperf/traceperf-tester
NAME                                   SYNCED   READY   STATUS
TracePerf/traceperf-tester (default)   True     True    Available
└─ XTracePerf/traceperf-tester-nxpvx   True     True    Available
   ├─ Object/object-0                  True     True    Available
   ├─ Object/object-1                  True     True    Available
   <omitted>
   └─ Object/object-99                 True     True    Available
crossplane beta trace traceperf/traceperf-tester  0.15s user 0.05s system 1% cpu 18.082 total
```

## Local Development

The resources in this repo can be tested and iterated on locally with the
following command to test using `render`:
```
crossplane beta render xr-tester.yaml composition.yaml functions.yaml -r
```