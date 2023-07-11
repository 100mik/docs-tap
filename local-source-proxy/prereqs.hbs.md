# Prerequisites for Local Source Proxy

You need the following prerequisites before you can install Local Source Proxy (LSP):

- A registry server with a repository capable of accepting and hosting OCI artifacts, such as Google
  Artifact Registry, JFrog Artifactory, Harbor, and so on.

- A secret with sufficient privileges to push and pull artifacts from that repository

The rest of this topic tells you how to obtain these prerequisites.

Using Tanzu CLI
: All registries except ECR can use the following code:

    ```console
    tanzu secret registry add lsp-push-credentials \
    --username USERNAME-VALUE --password PASSWORD-VALUE \
    --server REGISTRY-SERVER \
    --namespace tap-install --yes
    ```

Declarative syntax
: For declarative syntax:

    ```yaml
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: lsp-push-credentials
      namespace: tap-install
    type: kubernetes.io/dockerconfigjson
    stringData:
      .dockerconfigjson: BASE64-ENCODED-DOCKER-CONFIG-JSON
    ```

    `dockerconfigjson` structure is as follows:

    ```json
    {"auths":{"REGISTRY-SERVER":{"username":"USERNAME-VALUE","password": "PASSWORD-VALUE"}}}
    ```

    If you're using the Tanzu Application Platform GitOps installer using SOPS, after
    using SOPS to encrypt the secret put the secret in the
    `clusters/CLUSTER-NAME/cluster-config/config/lsp` directory in your GitOps repository.

    If you're using the Tanzu Application Platform GitOps installer using ESO, create a secret as
    follows:

    ```json
    #@ load("@ytt:data", "data")
    #@ load("@ytt:json", "json")

    #@ def config():
    #@  return {
    #@    "auths": {
    #@      data.values.tap_value.{path-to-registry-host}: {
    #@       "username": data.values.tap_values.{path-to-registry-username},
    #@       "password": data.values.tap_values.{path-to-registry-password}
    #@      }
    #@    }
    #@  }
    #@ end
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: lsp-push-credentials
      namespace: tap-install
    type: kubernetes.io/dockerconfigjson
    stringData:
      .dockerconfigjson: #@ json.encode(config())
    ```

The procedure you use to obtain a secret with sufficient privileges depends on whether your registry
is Elastic Container Registry (ECR) or something else.

Using AWS
: If you're using Elastic Container Registry as your registry, you require an AWS IAM role ARN
  that possesses the necessary privileges to push and pull artifacts to the ECR repository. If such
  a role does not exist, you can create one by following the steps in this topic.

Using a secret with pull privileges only
: You can use a secret with only pull privileges if you prefer to have a dedicated credential with a
  least-privilege policy, specifically for downloading artifacts instead of reusing credentials with
  higher privileges.

  The secret containing this credential is distributed across developer namespaces by using the
  Secretgen `SecretExport` resource. Namespace Provisioner automatically imports it to the developer
  namespace. However, for development purposes, you can skip this step and use the same secret for
  both pushing and pulling artifacts.

  To use a secret with pull privileges only, run:

    ```console
    # For all registries except ECR
    tanzu secret registry add lsp-pull-credentials \
    --username USERNAME-VALUE --password 'PASSWORD-VALUE' \
    --server REGISTRY-SERVER \
    --namespace tap-install --yes
    ```
