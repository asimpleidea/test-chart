# CN-WAN Operator Helm Chart

![Version: 1.0.29](https://img.shields.io/badge/Version-1.0.29-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: v0.5.0](https://img.shields.io/badge/AppVersion-v0.5.0-informational?style=flat-square)

Register and manage your Kubernetes Services to a Service Registry.

This is 1.0.29.

## Useful links

[Project's Homepage](https://github.com/CloudNativeSDWAN/cnwan-operator)
[Documentation](https://github.com/CloudNativeSDWAN/cnwan-operator#documentation)

**Sources**:

* <https://github.com/CloudNativeSDWAN/cnwan-operator>

## Requirements

**Kubernetes Version**: `>= 1.11.0-0`

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | etcd | > 6.2.0 |

The `bitnami/etcd` dependency is only required if you intend to install the
operator together with etcd. Otherwise, i.e. if you have etcd already installed
or do not want to use etcd, it won't be needed.

## Values table

The behavior of the operator and the ways it connects to and manages a service
registry can be fine tuned by setting the parameters of `operator` according
to your needs -- e.g. by setting a value to `operator.namespacePolicy` -- and
those of the service registry according to the one you are going to use -- e.g.
`operator.etcd` or `operator.googleServiceDirectory`.

Values that start with `etcd` -- e.g. `etcd.auth.rbac.enabled` will configure
the etcd cluster that will be installed along with the operator in case you set
`operator.etcd.install` to `true`, otherwise they will be ignored.

We highly recommend you to read the
[official documentation](https://github.com/CloudNativeSDWAN/cnwan-operator/blob/master/docs/configuration.md)
to better understand the operator's settings.

Finally, look at [examples](#examples) to learn more.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| etcd | object | See each value below. | Options about the etcd to install along with the operator, in case `operator.etcd.install` is true. |
| etcd.auth.rbac.allowNoneAuthentication | bool | `false` | Whether to allow connections to etcd as an unauthenticated user. |
| etcd.auth.rbac.enabled | bool | `true` | Whether to enable RBAC for etcd. |
| etcd.auth.rbac.existingSecret | string | The secret will be automatically deployed by the chart and | The name of the existing secret on your cluster where to get the root password. Please **do not** modify unless you know what you are doing. this value will be created internally. |
| etcd.fullnameOverride | string | `"service-registry"` | The name to give to the etcd installation if `operator.etcd.install` is true. |
| etcd.service.type | string | `"ClusterIP"` | The Kubernetes service type to use to expose the etcd installation. |
| image.pullPolicy | string | `"Always"` | The `pullPolicy` for this container image. |
| image.pullSecrets | list | The default repository is public, so no secret will be needed. | Secrets to use to pull the image, in case the provided repository is private. |
| image.registry | string | `"ghcr.io"` | The registry where to get the container image. |
| image.repository | string | `"cloudnativesdwan/cnwan-operator"` | The repository where to get the container image inside the registry specified above. |
| image.tag | string | The most recent version will be used. | The tag for the container image. |
| operator.cloudMetadata.network | string | If empty it will be omitted from settings. | Whether to register the current network name (VPC) among a service's metadata. Set auto if you are running in GKE to retrieve this value automatically. Omit if you don't need this. |
| operator.cloudMetadata.subNetwork | string | If empty it will be omitted from settings. | Whether to register the current sub-network name among a service's metadata. Set auto if you are running in GKE to retrieve this value automatically. Omit if you don't need this. |
| operator.etcd | object | Read each value below. | Options about etcd. |
| operator.etcd.endpoints | list | If `operator.etcd.install` is true, this will be filled | A list of endpoints where your etcd nodes are serving from. automatically, otherwise it will be `["etcd.etcd:2379"]`. |
| operator.etcd.install | bool | `false` | Whether to install etcd in the current namespace. Set this to false or omit this if you plan to install it yourself or already have it running somewhere. |
| operator.etcd.prefix | string | `"/service-registry"` | The prefix that all keys will share. |
| operator.etcd.username | string | `"root"` | The password to use for the username provided in `operator.etcd.username`. |
| operator.googleServiceDirectory | object | Read each value below. | Options about service directory. Will be ignored if `operator.serviceRegistry` is not `gcpServiceDirectory`. |
| operator.googleServiceDirectory.defaultRegion | string | If not provided, will be omitted from settings. | The default region to use for Service Directory. You *must* specify this, unless you are running in GKE *and* want to use the current region, in which case you can just omit this. |
| operator.googleServiceDirectory.projectID | string | If not provided, will be omitted from settings. | The GCP project ID to use for Service Directory. You *must* specify this, unless you are running in GKE *and* want to use the current project, in which case you can just omit this. |
| operator.googleServiceDirectory.serviceAccount | string | **DO NOT** set explicitly: see [Examples](#examples). | The path to the service account to use to authenticate to GCP. **DO NOT** set this explicitly, but rather supply this information during installation with `--set-file`. See [Examples](#examples) section. |
| operator.namespaceListPolicy | string | `"allowlist"` | The namespace list policy. Must be either `allowlist` or `blocklist`. [Learn more here](https://github.com/CloudNativeSDWAN/cnwan-operator/blob/master/docs/configuration.md) |
| operator.serviceAnnotations | list | **At least one item must be provided.** | A list of service annotations that must be watched. **An error will occur if this is empty.** [Learn more here](https://github.com/CloudNativeSDWAN/cnwan-operator/blob/master/docs/configuration.md) |
| operator.serviceRegistry | string | Required: will throw an error if not provided or invalid. | The service registry to use to register your resources. Must be either `etcd` or `ServiceDirectory`. |

***

## Examples

The following are quickstarts/examples to demonstrate how to deploy the
operator and configure it to use a certain service registry.

Refer to the [values table](#values-table) above for the meaning of the values
that have been set as `--set` or `--set-file`.

Please note that you can group all settings together with one `--set` only by
separating all values with commas, but in the following examples we use
`--set` for each separate setting to make the example more reader-friendly.

### Usage with Google Service Directory

This will deploy the operator to your cluster and will configure it to use
Google Service Directory.

It will need a valid
[service account](https://cloud.google.com/iam/docs/service-accounts)
in order to log in to GCP correctly: once you have located it, set its path
with `--set-file operator.googleServiceDirectory.serviceAccount <path>`. In
the following example, we assume its path is `$HOME/Desktop/service-account.json`.

```bash
kubectl create ns cnwan-operator-system
helm install cnwan-operator CloudNativeSDWAN/cnwan-operator \
--set operator.namespaceListPolicy=allowlist \
--set operator.serviceAnnotations="{traffic-profile}" \
--set operator.serviceRegistry=ServiceDirectory \
--set operator.googleServiceDirectory.projectID=my-project-dg502 \
--set operator.googleServiceDirectory.defaultRegion=us-west2 \
--set-file operator.googleServiceDirectory.serviceAccount=$HOME/Desktop/service-account.json \
-n cnwan-operator-system
```

If you are going to deploy this on *GKE* and want to use your default project
and you want your services to be registered on the same region as your *GKE*
cluster, you may omit `defaultRegion` and `projectID` values, like so:

```bash
kubectl create ns cnwan-operator-system
helm install cnwan-operator CloudNativeSDWAN/cnwan-operator \
--set operator.namespaceListPolicy=allowlist \
--set operator.serviceAnnotations="{traffic-profile}" \
--set operator.serviceRegistry=ServiceDirectory \
--set-file operator.googleServiceDirectory.serviceAccount=$HOME/Desktop/service-account.json \
-n cnwan-operator-system
```

### Usage with etcd

You may choose to deploy the operator along with a brand new etcd installation
and make it automatically configure the appropriate values for connecting and
managing etcd, or decide to only deploy the operator. In the latter case, you
may need to provide some values manually.

For brevity, some of the values here have been omitted and their default values
will apply in that case. Please refer to the [values table](#values-table)
above if you want to add other values and to know what they mean.

#### Installing the operator and etcd

To deploy the operator and etcd together, you can run

```bash
kubectl create ns cnwan-operator-system
helm install cnwan-operator CloudNativeSDWAN/cnwan-operator \
--set operator.serviceAnnotations="{traffic-profile}" \
--set operator.serviceRegistry=etcd \
--set operator.etcd.install=true \
-n cnwan-operator-system
```

Now wait a few minutes until service registry gets into ready state

```bash
watch kubectl get pods -n cnwan-operator-system
```

#### Installing the operator without etcd

This is the way to install the operator in case you have etcd already running
somewhere or you just want to install it and managing. Let's suppose your etcd
installation can be reached via DNS name `my-etcd.etcd` on port `8080` and the
password is `my-PWD`.

```bash
kubectl create ns cnwan-operator-system
helm install cnwan-operator CloudNativeSDWAN/cnwan-operator \
--set operator.serviceAnnotations="{traffic-profile}" \
--set operator.serviceRegistry=etcd \
--set operator.etcd.endpoints="{my-etcd.etcd:8080}" \
--set operator.etcd.password="my-PWD" \
-n cnwan-operator-system
```

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| CloudNativeSDWAN | contact@cisco.com | https://github.com/CloudNativeSDWAN/cnwan-operator/blob/master/OWNERS.md |
