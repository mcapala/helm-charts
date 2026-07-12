

# setup-istio

  [![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/openshift-bootstraps)](https://artifacthub.io/packages/search?repo=openshift-bootstraps)
  [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
  [![Lint and Test Charts](https://github.com/tjungbauer/helm-charts/actions/workflows/lint_and_test_charts.yml/badge.svg)](https://github.com/tjungbauer/helm-charts/actions/workflows/lint_and_test_charts.yml)
  [![Release Charts](https://github.com/tjungbauer/helm-charts/actions/workflows/release.yml/badge.svg)](https://github.com/tjungbauer/helm-charts/actions/workflows/release.yml)

  ![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square)

 

  ## Description

  Installs OpenShift Service Mesh 3 (servicemeshoperator3), creates istio-system/istio-cni/istio-csr namespaces, and configures IstioCNI, IstioCSR (cert-manager istio-csr agent) and the Istio control plane with mesh-wide mTLS.

This Helm chart deploys and configures **OpenShift Service Mesh 3** (sail-operator based). It provides a declarative way to manage:

- **Operator installation** - `servicemeshoperator3` via OLM Subscription (helper-operator) with CSV readiness gating (helper-status-checker)
- **Namespaces** - `istio-system`, `istio-cni` and `istio-csr` (list-driven, fully configurable)
- **IstioCNI** - the Istio CNI node agent
- **IstioCSR** - the cert-manager istio-csr agent, delegating workload and istiod certificate signing to a cert-manager `Issuer`
- **Istio** - the control plane, including `meshConfig.discoverySelectors` and `global.caAddress` pointing at istio-csr
- **PeerAuthentication** - mesh-wide mTLS policy (STRICT by default)

## Prerequisites

### Required

- **OpenShift** 4.19+
- **Helm** 3.0+
- **cert-manager Operator for Red Hat OpenShift** 1.15+ (provides the `IstioCSR` CRD) if `istiocsr.enabled` is set
- A cert-manager `Issuer` (default: `istio-ca` in `istio-system`) holding the mesh intermediate CA

## Mesh membership

Namespaces join the mesh via `meshConfig.discoverySelectors` (items are ORed):

- explicit namespace names (`istio-system`, `istio-cni`, `istio-csr`, `openshift-ingress` by default)
- any namespace labeled `istio-discovery: enabled`

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

Source code: https://github.com/mcapala/helm-charts/tree/main/charts/setup-istio

## Parameters

Verify the subcharts for additional settings:

* [tpl](https://github.com/tjungbauer/helm-charts/tree/main/charts/tpl)
* [helper-operator](https://github.com/tjungbauer/helm-charts/tree/main/charts/helper-operator)
* [helper-status-checker](https://github.com/tjungbauer/helm-charts/tree/main/charts/helper-status-checker)

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| helper-operator.operators.servicemesh-operator3.enabled | bool | `true` | Install the operator yes/no. |
| helper-operator.operators.servicemesh-operator3.namespace.create | bool | `false` | Do not create the namespace, it always exists. |
| helper-operator.operators.servicemesh-operator3.namespace.name | string | `"openshift-operators"` | Namespace the Subscription is created in. openshift-operators already exists. |
| helper-operator.operators.servicemesh-operator3.operatorgroup.create | bool | `false` | openshift-operators ships a global OperatorGroup - do not create another one. |
| helper-operator.operators.servicemesh-operator3.operatorgroup.notownnamespace | bool | `true` |  |
| helper-operator.operators.servicemesh-operator3.subscription.approval | string | `"Automatic"` | InstallPlan approval strategy (Automatic or Manual). |
| helper-operator.operators.servicemesh-operator3.subscription.channel | string | `"stable"` | Subscription channel. |
| helper-operator.operators.servicemesh-operator3.subscription.operatorName | string | `"servicemeshoperator3"` | Package name of the operator. |
| helper-operator.operators.servicemesh-operator3.subscription.source | string | `"redhat-operators"` | Catalog source. |
| helper-operator.operators.servicemesh-operator3.subscription.sourceNamespace | string | `"openshift-marketplace"` | Namespace of the catalog source. |
| helper-operator.operators.servicemesh-operator3.syncwave | string | `"0"` | Syncwave for the operator installation. |
| helper-status-checker.checks[0] | object | `{"namespace":{"name":"openshift-operators"},"operatorName":"servicemeshoperator3","serviceAccount":{"name":"status-checker-servicemesh"}}` | String the CSV name is grepped for (CSV: servicemeshoperator3.vX.Y.Z). |
| helper-status-checker.checks[0].namespace.name | string | `"openshift-operators"` | Namespace the CSV appears in. |
| helper-status-checker.checks[0].serviceAccount.name | string | `"status-checker-servicemesh"` | ServiceAccount used by the checker Job. |
| helper-status-checker.enabled | bool | `true` | Enable the status checker Job. |
| istio.additionalAnnotations | object | {} | Additional annotations for the resource. |
| istio.additionalLabels | object | {} | Additional labels for the resource. |
| istio.discoverySelectors | list | see values.yaml | meshConfig.discoverySelectors: namespaces that are part of the mesh. Items are ORed. Add application namespaces by labeling them istio-discovery=enabled. |
| istio.enabled | bool | `true` | Deploy the Istio (control plane) custom resource yes/no. |
| istio.name | string | `"default"` | Name of the Istio resource. |
| istio.namespace | string | `"istio-system"` | Namespace istiod is deployed to. |
| istio.syncwave | string | `"4"` | Syncwave: after IstioCNI and IstioCSR. |
| istio.updateStrategy | object | `{"type":"InPlace"}` | Update strategy for the control plane (InPlace or RevisionBased). |
| istio.values | object | see values.yaml | Free-form .spec.values (sail-operator helm values). discoverySelectors above are merged into values.meshConfig. global.caAddress points istiod and workloads to the istio-csr agent (disable together with istiocsr.enabled). |
| istio.version | string | "" | Optional: pin the control plane version (e.g. v1.26.4). Empty string uses the operator default. |
| istiocni.additionalAnnotations | object | {} | Additional annotations for the resource. |
| istiocni.additionalLabels | object | {} | Additional labels for the resource. |
| istiocni.enabled | bool | `true` | Deploy the IstioCNI custom resource yes/no. |
| istiocni.name | string | `"default"` | Name of the IstioCNI resource. |
| istiocni.namespace | string | `"istio-cni"` | Namespace the CNI DaemonSet runs in. |
| istiocni.syncwave | string | `"3"` | Syncwave: after the operator is ready. |
| istiocni.values | object | {} | Optional: free-form .spec.values passed to the IstioCNI resource (sail-operator helm values). |
| istiocni.version | string | "" | Optional: pin the Istio CNI version (e.g. v1.26.4). Empty string uses the operator default. |
| istiocsr.additionalAnnotations | object | {} | Additional annotations for the resource. |
| istiocsr.additionalLabels | object | {} | Additional labels for the resource. |
| istiocsr.enabled | bool | `true` | Deploy the IstioCSR custom resource (cert-manager operator must be installed). Delegates workload/istiod certificate signing to cert-manager. |
| istiocsr.istioCSRConfig | object | see values.yaml | istioCSRConfig rendered 1:1 into .spec.istioCSRConfig. The issuerRef must point to a cert-manager Issuer holding the mesh intermediate CA. |
| istiocsr.name | string | `"default"` | Name of the IstioCSR resource. |
| istiocsr.namespace | string | `"istio-csr"` | Namespace the istio-csr agent runs in. |
| istiocsr.syncwave | string | `"3"` | Syncwave: parallel to IstioCNI, before the Istio control plane. |
| namespaces | list | see values.yaml | List of namespaces this chart shall create. Used for istio-system, istio-cni and istio-csr. |
| namespaces[0] | object | `{"additionalAnnotations":{},"additionalLabels":{},"descr":"OpenShift Service Mesh control plane","enabled":true,"name":"istio-system","syncwave":"0"}` | Name of the namespace. |
| namespaces[0].additionalAnnotations | object | {} | Additional annotations to add to the namespace as key: value pairs. |
| namespaces[0].additionalLabels | object | {} | Additional labels to add to the namespace as key: value pairs. |
| namespaces[0].descr | string | `"OpenShift Service Mesh control plane"` | Optional: OpenShift description of the namespace. |
| namespaces[0].enabled | bool | `true` | Create this namespace yes/no. |
| namespaces[0].syncwave | string | 0 | Syncwave for the namespace creation. |
| peerauthentication.additionalAnnotations | object | {} | Additional annotations for the resource. |
| peerauthentication.additionalLabels | object | {} | Additional labels for the resource. |
| peerauthentication.enabled | bool | `true` | Create a mesh-wide PeerAuthentication in the control plane namespace. |
| peerauthentication.mtls.mode | string | `"STRICT"` | mTLS mode for the whole mesh (STRICT, PERMISSIVE or DISABLE). |
| peerauthentication.name | string | `"default"` | Name of the PeerAuthentication resource. |
| peerauthentication.namespace | string | `"istio-system"` | Namespace: control plane namespace makes the policy mesh-wide. |
| peerauthentication.syncwave | string | `"5"` | Syncwave: after the control plane. |

## Example: cert-manager integration (istio-csr)

With `istiocsr.enabled: true` the chart deploys the `IstioCSR` custom resource and points the control plane at the agent:

```yaml
istiocsr:
  enabled: true
  namespace: istio-csr
  istioCSRConfig:
    certManager:
      issuerRef:
        name: istio-ca
        kind: Issuer
        group: cert-manager.io
    istiodTLSConfig:
      trustDomain: cluster.local
    istio:
      namespace: istio-system

istio:
  values:
    global:
      caAddress: cert-manager-istio-csr.istio-csr.svc:443
```

The referenced `Issuer` (and its CA `Certificate`) must be created separately, e.g. by the cert-manager chart.

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
