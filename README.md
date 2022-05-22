# Introduction

This repo provides a few examples of automating day two operations on OpenShift with Ansible.

## Use Cases

* Labeling Nodes
* Creating TLS Secrets
* Configuring TLS on API/Ingress Endpoints
* Scaling Scaling Ingress Controller
* Deploying ODF

## Getting Started

To get started with these examples, clone this git repository on your local machine by running the following:

```shell
git clone https://github.com/nasx/demo-temp.git
```

Move to the root of the repository (you should see `ansible.cfg` in this directory). These playbooks leverage the use of Ansible collections. To install the requisite collections, run the following command:

```shell
ansible-galaxy collection install -r collections/requirements.yaml
```

The playbooks in this cluster will require the KUBECONFIG environment variable to be set. Export the environment variable and ensure you can connect to the cluster using the `kubectl`/`oc` commands.

## Example 1 - Labeling Nodes

Modify the `nodes` variable in the `01-label-nodes.yaml` playbook to suite your environment. The nodes variable follows the following format:

```yaml
nodes:
  - name: "node1.fqdn.com"
    labels:
      key1: value1
      key2: value2
  - name: "node2.fqdn.com"
  	labels:
  	  key1: value1
  	  key2: value2
```

NOTE: Make sure the "names" of the nodes match the output of `oc get nodes`.

Run the playbook using the following:

```shell
ansible-playbook 01-label-nodes.yaml
```

## Example 2 - TLS Secrets

This playbook requires the `tls_secrets` variable to be set. For each secret we will need the TLS keypair (private key and certificate) as well as the CA certificate from the issuing certificate authority.

The `tls_secrets` variable has the following format:

```yaml
tls_secrets:
  cluster_name:
    - name: secret1
      namespace: my-namespace
      crt: <b64 encoded certificates>
      key: <b64 encoded private key>
```

Variables:
* `cluster_name` - Name of the cluster we are applying secrets to.
* `name` - Name of the k8s Secret
* `namespace` - Namespace the k8s Secret will be created in
* `crt` - Base64 encoded PEM certificate and (appended) CA certificate
* `key` - Base64 encoded PEM key

This variable should be stored in an Ansible vault. 

To create the value for the `crt` variable, run the following command:

```shell
cat certificate.pem ca.pem | base64 -w0
```

To create the value for the `key` variable, run the following command:

```shell
cat key.pem | base64 -w0
```

Run the playbook as follows:

```shell
ansible-playbook --ask-vault-pass -e @local/tls-secrets.yaml -e cluster_name=my_cluster 02-tls-secrets.yaml
```

## Example 3 - Ingress Controller

This example will adjust the number of Ingress controller replicas and update the TLS certificate. The default variables for this role are:

```yaml
ingress_name: default
ingress_replicas: 3
ingress_tls_secret_name: wildcard-cert
```

Override these values as applicable to your environment by creating a separate variables file or overriding them on the CLI as shown below:

```shell
ansible-playbook -e ingress_tls_secret_name=my-cert 03-ingress.yaml
```

## Example 4 - API Server

This example will update the TLS certificate for the API server. The default variables for this role are:

```yaml
api_tls_secret_name: api-cert
```

Override these values as applicable to your environment by creating a separate variables file or overriding them on the CLI as shown below:

```shell
ansible-playbook -e api_tls_secret_name=my-cert 04-api-server.yaml
```

## Example 5 - Deploy ODF

This role will deploy ODF in your environment. Make sure the ODF nodes are properly labeled in your cluster (`cluster.ocs.openshift.io/openshift-storage: ""`) before starting.

The default variables for this role are:

```yaml
odf_channel: stable-4.10
odf_installplan_approval: Automatic
odf_namespace: openshift-storage
odf_operatorgroup_name: openshift-storage-operatorgroup
odf_subscription_name: odf-operator
storagecluster_deviceset_name: odf-deviceset-thin
storagecluster_name: odf-storagecluster
storagecluster_osd_size: "2Ti"
storagecluster_storageclass_name: thin
```

Override these values as applicable to your environment by creating a separate variables file or overriding them on the CLI as shown in previous examples. Run the playbook as follows (be sure to add extra variables as applicable):

```shell
ansible-playbook 05-deploy-odf.yaml
```
