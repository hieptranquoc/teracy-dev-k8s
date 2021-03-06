# Cert Manager

[cert-manager][] is used for provisioning and managing TLS certificates in Kubernetes automatically.


## Enable cert-manager

- Configure the `teracy-dev-entry/config_override.yaml` file with the following similar content:


```yaml
teracy-dev-k8s:
  ansible:
    host_vars:
      cert_manager_enabled: "True"
```

  and then `$ vagrant reload --provision`.

- `$ kubectl -n cert-manager get pod` should show the similar following output:

```bash
NAME                            READY   STATUS    RESTARTS   AGE
cert-manager-695f7b5bdc-pchmn   1/1     Running   0          14h
```



## Use the generated key-pair from `teracy-dev-certs` to create a CA issuer

- Make sure to enable and configure `teracy-dev-certs` by following https://github.com/teracyhq-incubator/teracy-dev-entry-k8s/blob/master/config_default.yaml#L50


- Use the key-pair of `k8s-local-ca.key` and `k8s-local-ca.crt` to configure a CA issuer, for example:

  + Create the key-pair secret:

  ```
  $ cd workspace/certs
  $ kubectl -n cert-manager create secret tls ca-key-pair --key=k8s-local-ca.key --cert=k8s-local-ca.crt
  ```

  + Create the `ca-cluster-issuer` by executing the following commands:

  ```bash
  $ cd docs/cert-manager
  $ kubectl apply -f ca-cluster-issuer.yaml
  ```

- After that, `$ kubectl describe clusterissuers.certmanager.k8s.io ca-cluster-issuer` should show
  the following similar output:

```bash
Name:         ca-cluster-issuer
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  certmanager.k8s.io/v1alpha1
Kind:         ClusterIssuer
Metadata:
  Creation Timestamp:  2018-12-09T11:46:02Z
  Generation:          1
  Resource Version:    1161130
  Self Link:           /apis/certmanager.k8s.io/v1alpha1/clusterissuers/ca-cluster-issuer
  UID:                 01161972-fba8-11e8-8eea-08002781145e
Spec:
  Ca:
    Secret Name:  ca-key-pair
Status:
  Conditions:
    Last Transition Time:  2018-12-09T11:46:02Z
    Message:               Signing CA verified
    Reason:                KeyPairVerified
    Status:                True
    Type:                  Ready
Events:                    <none>
```

You can now use the `ca-cluster-issuer` to generate any certificates by following the docs from
https://cert-manager.readthedocs.io/en/latest/tutorials/ca/creating-ca-issuer.html.


Notes:

- The `ca-key-pair` secret must be created within the `cert-manager` namespace due to this:
  https://github.com/jetstack/cert-manager/issues/650 but `$ vagrant reload --provision` everytime
  will delete this created secret. There is a workaround that you need to comment the `cert-manager`
  configuration on the `teracy-dev-entry/config_override.yaml` as follows:

  ```yaml
  teracy-dev-k8s:
    ansible:
      host_vars: {} # need this empty {} config if host_vars has no values
        # comment cert_manager_enabled here so that ca-key-pair secret will not be deleted (workaround)
        # cert_manager_enabled: "True"
  ```


## References

- https://cert-manager.readthedocs.io/en/latest/index.html

- https://stackoverflow.com/questions/2957742/how-to-convert-pkcs8-formatted-pem-private-key-to-the-traditional-format

- https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key


[cert-manager]: https://github.com/jetstack/cert-manager
