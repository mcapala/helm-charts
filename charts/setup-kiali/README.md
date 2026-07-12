

# setup-kiali

  [![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/openshift-bootstraps)](https://artifacthub.io/packages/search?repo=openshift-bootstraps)
  [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
  [![Lint and Test Charts](https://github.com/tjungbauer/helm-charts/actions/workflows/lint_and_test_charts.yml/badge.svg)](https://github.com/tjungbauer/helm-charts/actions/workflows/lint_and_test_charts.yml)
  [![Release Charts](https://github.com/tjungbauer/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/tjungbauer/helm-charts/actions/workflows/release.yml)

  ![Version: 1.0.3](https://img.shields.io/badge/Version-1.0.3-informational?style=flat-square)

 

  ## Description

  Installs the Kiali operator (kiali-ossm) and a Kiali instance integrated with OpenShift platform monitoring (thanos-querier), including ServiceMonitor/PodMonitor resources for istiod and Envoy proxies.

This Helm chart deploys and configures **Kiali** for OpenShift Service Mesh 3, integrated with OpenShift platform monitoring. It provides a declarative way to manage:

- **Operator installation** - `kiali-ossm` via OLM Subscription (helper-operator) with CSV readiness gating (helper-status-checker)
- **Kiali** - a Kiali instance wired to the platform Prometheus (`thanos-querier`) with bearer-token auth, mesh namespace discovery selectors and both Gateway API classes (`istio`, `openshift-default`)
- **ServiceMonitor / PodMonitor** - scrape configs for istiod and Envoy sidecar metrics via User Workload Monitoring, per namespace (list-driven)

Based on the OpenShift monitoring integration described in the
[openshift-gateway-mesh-integration monitoring blog](https://github.com/openlab-red/openshift-gateway-mesh-integration/blob/main/docs/blogs/monitoring.md).

## Prerequisites

### Required

- **OpenShift** 4.19+
- **Helm** 3.0+
- **OpenShift Service Mesh 3** control plane (e.g. via the `setup-istio` chart) — the Kiali resource lives in the control plane namespace
- **User Workload Monitoring** enabled (`enableUserWorkload: true` in the `cluster-monitoring-config` ConfigMap), so the ServiceMonitor/PodMonitor resources are scraped

## Dependencies

This chart has the following dependencies:

| Repository | Name | Version |
|------------|------|---------|
| https://charts.stderr.at/ | helper-operator | ~1.0.0 |
| https://charts.stderr.at/ | helper-status-checker | ~4.0.0 |
| https://charts.stderr.at/ | tpl | ~1.0.0 |

It is best used with a full GitOps approach such as Argo CD does. For example, https://github.com/tjungbauer/openshift-clusterconfig-gitops

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| mcapala |  |  |

## Sources
Source:
* <https://github.com/mcapala/helm-charts>

Source code: https://github.com/mcapala/helm-charts/tree/main/charts/setup-kiali

## Parameters

Verify the subcharts for additional settings:

* [tpl](https://github.com/tjungbauer/helm-charts/tree/main/charts/tpl)
* [helper-operator](https://github.com/tjungbauer/helm-charts/tree/main/charts/helper-operator)
* [helper-status-checker](https://github.com/tjungbauer/helm-charts/tree/main/charts/helper-status-checker)

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| helper-operator.operators.kiali-operator.enabled | bool | `true` | Install the operator yes/no. |
| helper-operator.operators.kiali-operator.namespace.create | bool | `false` | Do not create the namespace, it always exists. |
| helper-operator.operators.kiali-operator.namespace.name | string | `"openshift-operators"` | Namespace the Subscription is created in. openshift-operators always exists. |
| helper-operator.operators.kiali-operator.operatorgroup.create | bool | `false` | openshift-operators ships a global OperatorGroup - do not create another one. |
| helper-operator.operators.kiali-operator.operatorgroup.notownnamespace | bool | `true` |  |
| helper-operator.operators.kiali-operator.subscription.approval | string | `"Automatic"` | InstallPlan approval strategy (Automatic or Manual). |
| helper-operator.operators.kiali-operator.subscription.channel | string | `"stable"` | Subscription channel. |
| helper-operator.operators.kiali-operator.subscription.operatorName | string | `"kiali-ossm"` | Package name of the operator. |
| helper-operator.operators.kiali-operator.subscription.source | string | `"redhat-operators"` | Catalog source. |
| helper-operator.operators.kiali-operator.subscription.sourceNamespace | string | `"openshift-marketplace"` | Namespace of the catalog source. |
| helper-operator.operators.kiali-operator.syncwave | string | `"0"` | Syncwave for the operator installation. |
| helper-status-checker.checks[0] | object | `{"namespace":{"name":"openshift-operators"},"operatorName":"kiali-operator","serviceAccount":{"name":"status-checker-kiali"}}` | String the CSV name is grepped for (CSV: kiali-operator.vX.Y.Z). |
| helper-status-checker.checks[0].namespace.name | string | `"openshift-operators"` | Namespace the CSV appears in. |
| helper-status-checker.checks[0].serviceAccount.name | string | `"status-checker-kiali"` | ServiceAccount used by the checker Job. |
| helper-status-checker.enabled | bool | `true` | Enable the status checker Job. |
| kiali.additionalAnnotations | object | {} | Additional annotations for the resource. |
| kiali.additionalLabels | object | {} | Additional labels for the resource. |
| kiali.auth | object | {} | Optional: authentication strategy (openshift, anonymous, token). Empty = operator default (openshift). |
| kiali.deployment.cluster_wide_access | bool | `true` | Give Kiali access to all namespaces (required with discovery selectors). |
| kiali.deployment.discovery_selectors | list | see values.yaml | Namespaces Kiali watches (rendered into deployment.discovery_selectors.default, items are ORed). Should mirror the Istio meshConfig.discoverySelectors. |
| kiali.deployment.view_only_mode | bool | `false` | Run Kiali in view-only mode (no changes via UI). |
| kiali.enabled | bool | `true` | Deploy the Kiali custom resource yes/no. |
| kiali.external_services.grafana.enabled | bool | `false` | Grafana integration. Explicitly disabled by default - Kiali's internal default is enabled=true, which probes grafana.istio-system:3000 and logs connection warnings. |
| kiali.external_services.istio.gateway_api_classes | list | see values.yaml | Gateway API classes shown by Kiali. Registering istio AND openshift-default avoids KIA1504. |
| kiali.external_services.perses | object | {} | Optional: Perses dashboard integration (enabled, internal_url, external_url, url_format, project), e.g. deployed via the Cluster Observability Operator. With the Cluster Observability Operator set url_format "openshift", leave internal_url unset and point external_url at the cluster console URL without any additional path. |
| kiali.external_services.prometheus.auth.type | string | `"bearer"` | Authentication type used against Prometheus (bearer for OpenShift platform monitoring). |
| kiali.external_services.prometheus.auth.use_kiali_token | bool | `true` | Use the Kiali ServiceAccount token as the bearer token. |
| kiali.external_services.prometheus.thanos_proxy.enabled | bool | `true` | The platform thanos-querier speaks the Thanos API - must be enabled. |
| kiali.external_services.prometheus.url | string | `"https://thanos-querier.openshift-monitoring.svc.cluster.local:9091"` | URL of the platform Prometheus (thanos-querier). |
| kiali.external_services.tracing | object | {} | Optional: distributed tracing integration, e.g. Tempo (enabled, provider, internal_url, external_url, use_grpc, auth.type, auth.use_kiali_token). With TempoStack multitenancy use the gateway tenant URL and bearer auth with the Kiali ServiceAccount token. |
| kiali.name | string | `"kiali"` | Name of the Kiali resource. |
| kiali.namespace | string | `"istio-system"` | Namespace of the Kiali resource (control plane namespace). |
| kiali.syncwave | string | `"3"` | Syncwave: after the operator is ready. |
| monitoring.enabled | bool | `true` | Create ServiceMonitor/PodMonitor resources so platform Prometheus (UWM) scrapes istiod and Envoy metrics. |
| monitoring.namespaces | list | `[{"enabled":true,"name":"istio-system"},{"enabled":false,"name":"openshift-ingress"}]` | Namespaces to create the monitors in. openshift-ingress becomes relevant when a Gateway API gateway runs there. |
| monitoring.podMonitor.interval | string | `"30s"` | Scrape interval. |
| monitoring.podMonitor.name | string | `"envoy-stats-monitor"` | Name of the Envoy proxy PodMonitor. |
| monitoring.podMonitor.path | string | `"/stats/prometheus"` | Metrics path exposed by the Envoy sidecar. |
| monitoring.podMonitor.relabelings | list | see values.yaml | Relabeling rules: keep istio-proxy containers, build __address__ for IPv4/IPv6, map namespace/pod labels. |
| monitoring.serviceMonitor.interval | string | `"30s"` | Scrape interval. |
| monitoring.serviceMonitor.name | string | `"istiod-monitor"` | Name of the istiod ServiceMonitor. |
| monitoring.serviceMonitor.port | string | `"http-monitoring"` | Metrics port name on the istiod service. |
| monitoring.serviceMonitor.selector | object | `{"matchLabels":{"istio":"pilot"}}` | Label selector matching the istiod service. |
| monitoring.syncwave | string | `"3"` | Syncwave for the monitors. |

## Monitoring resources

`monitoring.namespaces` drives where the istiod `ServiceMonitor` and Envoy `PodMonitor` are created.
The `openshift-ingress` entry is pre-defined but disabled — enable it once a Gateway API gateway
(with its own istiod and Envoy pods) runs there:

```yaml
monitoring:
  namespaces:
    - name: istio-system
      enabled: true
    - name: openshift-ingress
      enabled: true
```

## Installing the Chart

To install the chart with the release name `my-release`:

```console
helm install my-release tjungbauer/<chart-name>>
```

The command deploys the chart on the Kubernetes cluster in the default configuration.

## Uninstalling the Chart

To uninstall/delete the my-release deployment:

```console
helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.14.2](https://github.com/norwoodj/helm-docs/releases/v1.14.2)
