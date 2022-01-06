# Installing cert-manager and contour packages using PackageInstall CR

This document describes how to install cert-manager package and contour package from the Tanzu Application Platform package repository using `PackageInstall` CRs

* **cert-manager**:

  1. List version information for the package by running:

        ```
        tanzu package available list cert-manager.tanzu.vmware.com -n tap-install
        ```

        For example:

        ```
        $ tanzu package available list cert-manager.tanzu.vmware.com -n tap-install
        / Retrieving package versions for cert-manager.tanzu.vmware.com...
          NAME                           VERSION      RELEASED-AT
          cert-manager.tanzu.vmware.com  1.5.3+tap.1  2021-08-23T17:22:51Z
        ```

  2. Create a `cert-manager-rbac.yml` using below sample and Apply the config.


        ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: cert-manager-tap-install-cluster-admin-role
        rules:
        - apiGroups:
          - '*'
          resources:
          - '*'
          verbs:
          - '*'
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: cert-manager-tap-install-cluster-admin-role-binding
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: cert-manager-tap-install-cluster-admin-role
        subjects:
        - kind: ServiceAccount
          name: cert-manager-tap-install-sa
          namespace: tap-install
        ---
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: cert-manager-tap-install-sa
          namespace: tap-install
        ```

        For example:

        ```
        kubectl apply -f cert-manager-rbac.yml
        ```

  3. Create a `cert-manager-install.yml` using below sample and Apply the config.


        ```yaml
        apiVersion: packaging.carvel.dev/v1alpha1
        kind: PackageInstall
        metadata:
          name: cert-manager
          namespace: tap-install
        spec:
          serviceAccountName: cert-manager-tap-install-sa
          packageRef:
            refName: cert-manager.tanzu.vmware.com
            versionSelection:
              constraints: "VERSION-NUMBER"
              prereleases: {}
        ```

        Where
        - `VERSION-NUMBER` is the version of the package listed in step 1.


        For example:

        ```
        kubectl apply -f cert-manager-rbac.yml
        ```

  4. Verify the package install by running:

        ```
        tanzu package installed get cert-manager -n tap-install
        ```

       For example:

        ```
        $ tanzu package installed get cert-manager -n tap-install
        / Retrieving installation details for cert-manager...
        NAME:                    cert-manager
        PACKAGE-NAME:            cert-manager.tanzu.vmware.com
        PACKAGE-VERSION:         1.5.3+tap.1
        STATUS:                  Reconcile succeeded
        CONDITIONS:              [{ReconcileSucceeded True  }]
        USEFUL-ERROR-MESSAGE:
        ```

        Verify that `STATUS` is `Reconcile succeeded`

        ```
        kubectl get deployment cert-manager -n cert-manager
        ```

        For example:

        ```
        $ kubectl get deploy cert-manager -n cert-manager
        NAME           READY   UP-TO-DATE   AVAILABLE   AGE
        cert-manager   1/1     1            1           2m18s
        ```

        Verify that `STATUS` is `Running`

* **contour**:

    1. List version information for the package by running:

        ```
        tanzu package available list contour.tanzu.vmware.com -n tap-install
        ```

        For example:

        ```
        $  tanzu package available list contour.tanzu.vmware.com -n tap-install
        - Retrieving package versions for contour.tanzu.vmware.com...
          NAME                      VERSION       RELEASED-AT
          contour.tanzu.vmware.com  1.18.2+tap.1  2021-10-05T00:00:00Z
        ```

    2. Create a `contour-rbac.yml` using the below sample and Apply the config.
        ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: contour-tap-install-cluster-admin-role
        rules:
        - apiGroups:
          - '*'
          resources:
          - '*'
          verbs:
          - '*'
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: contour-tap-install-cluster-admin-role-binding
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: contour-tap-install-cluster-admin-role
        subjects:
        - kind: ServiceAccount
          name: contour-tap-install-sa
          namespace: tap-install
        ---
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: contour-tap-install-sa
          namespace: tap-install
        ```

        For example:

        ```
        kubectl apply -f contour-rbac.yml
        ```
    
    3. Create a `contour-install.yml` using the samplebelow and apply the config.
    > **NOTE:** The below config will install contour package with default options. If you want to make changes to the default installation settings, go to step 4.

        ```yaml
        apiVersion: packaging.carvel.dev/v1alpha1
        kind: PackageInstall
        metadata:
          name: contour
          namespace: tap-install
        spec:
          serviceAccountName: tap-install-sa
          packageRef:
            refName: contour.tanzu.vmware.com
            versionSelection:
              constraints: "VERSION-NUMBER"
              prereleases: {}
        ```
        Where
        - `VERSION-NUMBER` is the version of the package listed in step 1.

    4. (Optional) Make changes to the default installation settings:

        1. Gather values schema by running:

            ```
            tanzu package available get contour.tanzu.vmware.com/1.18.2+tap.1 --values-schema -n tap-install
            ````

            For example:

            ```
            $ tanzu package available get contour.tanzu.vmware.com/1.18.2+tap.1 --values-schema -n tap-install
            | Retrieving package details for contour.tanzu.vmware.com/1.18.2+tap.1...
              KEY                                  DEFAULT               TYPE     DESCRIPTION

              certificates.duration                8760h                 string   If using cert-manager, how long the certificates should be valid for. If useCertManager is false, this field is ignored.
              certificates.renewBefore             360h                  string   If using cert-manager, how long before expiration the certificates should be renewed. If useCertManager is false, this field is ignored.
              contour.configFileContents           <nil>                 object   The YAML contents of the Contour config file. See https://projectcontour.io/docs/v1.18.2/configuration/#configuration-file for more information.
              contour.logLevel                     info                  string   The Contour log level. Valid options are info and debug.

              contour.replicas                     2                     integer  How many Contour pod replicas to have.

              contour.useProxyProtocol             false                 boolean  Whether to enable PROXY protocol for all Envoy listeners.

              envoy.hostPorts.enable               true                  boolean  Whether to enable host ports. If false, http and https are ignored.

              envoy.hostPorts.http                 80                    integer  If enable == true, the host port number to expose Envoy's HTTP listener on.

              envoy.hostPorts.https                443                   integer  If enable == true, the host port number to expose Envoy's HTTPS listener on.

              envoy.logLevel                       info                  string   The Envoy log level.

              envoy.service.annotations            <nil>                 object   Annotations to set on the Envoy service.

              envoy.service.aws.LBType             classic               string   AWS loadbalancer type.

              envoy.service.externalTrafficPolicy  Cluster               string   The external traffic policy for the Envoy service.

              envoy.service.nodePorts.http         <nil>                 integer  If type == NodePort, the node port number to expose Envoy's HTTP listener on. If not specified, a node port will be auto-assigned by Kubernetes.
              envoy.service.nodePorts.https        <nil>                 integer  If type == NodePort, the node port number to expose Envoy's HTTPS listener on. If not specified, a node port will be auto-assigned by Kubernetes.
              envoy.service.type                   NodePort              string   The type of Kubernetes service to provision for Envoy.

              envoy.terminationGracePeriodSeconds  300                   integer  The termination grace period, in seconds, for the Envoy pods.

              envoy.hostNetwork                    false                 boolean  Whether to enable host networking for the Envoy pods.

              infrastructure_provider              vsphere               string   The infrastructur in which to deploy Contour and Envoy. example:- vsphere, aws
              namespace                            tanzu-system-ingress  string   The namespace in which to deploy Contour and Envoy.
            ```

        2. Create a `contour-install.yaml` using the following sample as a guide:

            Sample `contour-install.yaml` for installation in aws public cloud with `LoadBalancer` services:

            ```yaml
            apiVersion: packaging.carvel.dev/v1alpha1
            kind: PackageInstall
            metadata:
              name: contour
              namespace: tap-install
            spec:
              serviceAccountName: contour-tap-install-sa
              packageRef:
                refName: contour.tanzu.vmware.com
                versionSelection:
                  constraints: 1.18.2+tap.1
                  prereleases: {}
              values:
              - secretRef:
                  name: contour-values
            ---
            apiVersion: v1
            kind: Secret
            metadata:
              name: contour-values
              namespace: tap-install
            stringData:
              values.yaml: |
                envoy:
                  service:
                    type: LoadBalancer
                infrastructure_provider: aws
            ```

            >**Note:** the LoadBalancer type is appropriate for most installations, but local clusters
              such as `kind` or `minikube` can fail to complete the package install if LoadBalancer
              services are not supported.

            >**Note:** Contour provides an Ingress implementation by default. If you have another Ingress
            implementation in your cluster, you must explicitly specify an
            [IngressClass](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-class)
            to select a particular implementation.

            >**Note:** [Cloud Native Runtimes](#install-cnr) programs Contour HTTPRoutes are based on the
            installed namespace. The default installation of CNR uses a single Contour to provide
            internet-visible services. You can install a second Contour instance with service type
            `ClusterIP` if you want to expose some services to only the local cluster. The second instance
            must be installed in a separate namespace. You must set the CNR value `ingress.internal.namespace` to
            point to this namespace.

    5. Install the package by running:

        ```
        kubectl apply -f contour-install.yaml
        ```


    6. Verify the package install by running:

        ```
        tanzu package installed get contour -n tap-install
        ```

        For example:

        ```
        $ tanzu package installed get contour -n tap-install
        / Retrieving installation details for contour...
        NAME:                    contour
        PACKAGE-NAME:            contour.tanzu.vmware.com
        PACKAGE-VERSION:         1.18.2+tap.1
        STATUS:                  Reconcile succeeded
        CONDITIONS:              [{ReconcileSucceeded True  }]
        USEFUL-ERROR-MESSAGE:
        ```

        Verify that `STATUS` is `Reconcile succeeded`

    7. Verify the installation:

        ```
        kubectl get po -n tanzu-system-ingress
        ```

        For example:

        ```
        $  kubectl get po -n tanzu-system-ingress
        NAME                       READY   STATUS    RESTARTS   AGE
        contour-857d46c845-4r6c5   1/1     Running   1          18d
        contour-857d46c845-p6bbq   1/1     Running   1          18d
        envoy-mxkjk                2/2     Running   2          18d
        envoy-qlg8l                2/2     Running   2          18d
        ```

        Ensure that all Pods are `Running` with all containers ready.

