---
title: "Configurations"
draft: false
---

### Plugins

OADP allows you to configure any set of Velero plugins you want. OADP includes
a default list of plugins that we support and optionally allows you to specify
any custom image if you have third-party plugin you'd like to install for
Velero. To learn more, see [plugin configuration](./plugins.md).

### Backup Storage Locations and Volume Snapshot Locations

Velero supports backup storage locations and volume snapshot locations from a
number of cloud providers (AWS, Azure and GCP). Please refer the section
[configure Backup Storage Locations and Volume Snapshot
Locations](./bsl_and_vsl.md). 

### Resource requests and limits customization

By default, the Velero deployment requests 500m CPU, 128Mi memory and sets a
limit of 1000m CPU, 512Mi. Customization of these resource requests and limits
may be performed using steps specified in the [Resource requests and limits
customization](./resource_req_limits.md) section.

### Use self-signed certificate

If you intend to use Velero with a storage provider that is secured by a
self-signed certificate, you may need to instruct Velero to trust that
certificate. See [Use self-signed certificate](./self_signed_certs.md)
section for details.

### Usage of Velero `--features` option
Some of the new features in Velero are released as beta features behind feature
flags which are not enabled by default during the Velero installation. In order
to provide `--features` flag values, you need to use the specify the flags
under `velero_feature_flags:` in the
`konveyor.openshift.io_v1alpha1_velero_cr.yaml` file during deployment.

Some of the usage instances of the `--features` flag are as follows:
- Enabling Velero plugin for CSI: To enable CSI plugin you need to add two 
  things in the `konveyor.openshift.io_v1alpha1_velero_cr.yaml` file during 
  deployment.
  - First, add `csi` under the `default_velero_plugins` 
  - Second, add `EnableCSI` under the `velero_feature_flags`
```
default_velero_plugins:
- csi
velero_feature_flags: EnableCSI
```
- Enabling Velero plugin for vSphere: To enable vSphere plugin you need to do 
  the following things in the `konveyor.openshift.io_v1alpha1_velero_cr.yaml` 
  file during deployment.
  - First, add `vsphere` under the `default_velero_plugins`
  - Second, add `EnableLocalMode` under the `velero_feature_flags`
  - Lastly, add the flag `use_upstream_images` and set it as `true`.
```
default_velero_plugins:
- vsphere
velero_feature_flags: EnableLocalMode
use_upstream_images: true
```
Note: The above is an example of installing the Velero plugin for vSphere in
`LocalMode` . Setting `EnableLocalMode` features flag is not always necessary
for the usage of vSphere plugin but the pre-requisites must be satisfied and
appropriate configuration must be applied, please refer [Velero plugin for
vSphere](https://github.com/vmware-tanzu/velero-plugin-for-vsphere) for more
details. Also, if you plan on using multiple feature flags at once, pass them
to `velero_feature_flags` as comma seperated values, for instance,
`velero_feature_flags: EnableLocalMode,EnableCSI`
