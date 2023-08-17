# retool-workflows-helm

[![Artifact HUB](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/retool)](https://artifacthub.io/packages/search?repo=retool)

This repository contains the official **Helm 3** chart for installing and configuring
Retool on Kubernetes. For full documentation on all the ways you can deploy
Retool on your own infrastructure, please see the [Setup
Guide](https://docs.retool.com/docs/setup-instructions).

## Prerequisites

* This chart requires **Helm 3.7**.
* A PostgreSQL database.
  * Persistent volumes are not reliable - we strongly recommend that a long-term
  installation of Retool host the database on an externally managed database (for example, AWS RDS).

## Usage
1. Add the Retool Helm repository:

        $ helm repo add retool https://charts.retool.com
        "retool" has been added to your repositories

2. Ensure you have access to the `retool-wf` chart:

        $ helm search repo retool/retool-wf
        NAME         	CHART VERSION	APP VERSION	DESCRIPTION                
        retool/retool-wf	4.0.0        	2.66.2     	A Helm chart for Kubernetes
3. Run this command `git clone https://github.com/tryretool/retool-workflows-helm.git`

4. Modify the `values.yaml` file:

* Set values for `config.encryptionKey` and `config.jwtSecret`. They should each be a different long, random string that you keep private. See our docs on [Environment Variables](https://docs.retool.com/docs/environment-variables) for more information on how they are used.

* Set `image.tag` with the version of Retool you want to install (i.e. a version in the format X.Y.Z). See our guide on [Retool Release Versions](https://docs.retool.com/docs/updating-retool-on-premise#retool-release-versions) to see our most recent version. **The minimum supported image for Retool Workflows is 2.108.4**

* Set `config.licenseKey` with your license key.

* To force Retool to send the auth cookies over HTTP, set `config.useInsecureCookies` to `true`. Leave the default value of `false` if you will use https to connect to the instance.

* Set `workflows.enabled` to `true` to enable Workflows.

5. Now you're all ready to install Retool:

        $ helm install my-retool retool/retool-wf -f values.yaml

## Additional Configuration

### Externalize database
Modify `values.yaml`:

* Disable the included postgresql chart by setting `postgresql.enabled` to `false`. Then specify your external database through the `config.postgresql.\*` properties at the top of the file. You will also need to specify these external database values in `retool-temporal-services-helm.server.config.persistence.{default & visibility}.sql`.

### gRPC
1. Create a `configMap` of the directory which contains your `proto` files.

        $ kubectl create configmap protos --from-file=<protos-path>

2. Modify `values.yaml`:

        extraVolumeMounts:
          - name: protos
          mountPath: /retool_backend/protos
          readOnly: true

        extraVolumes:
          - name: protos
          configMap:
            name: protos 

        env:
          PROTO_DIRECTORY_PATH=/retool_backend/protos

### Ingress
Modify `values.yaml`:

- Uncomment `ingress.hosts` and change `ingress.hosts.host` to be the hostname where you will access Retool.
- If you are implementing TLS for your Retool instance, uncomment `ingress.tls` and:
    - Specify the name of the SSL certificate to use as the value of `ingress.tls.secretName`.
    - Specify an array containing the hostname where you will access Retool (the same value you configured for `ingress.hosts.host`).
- If you are want more control over the the ingress path, for example in order to create a rediect from port 80 to 443 with aws-load-balancer-controller, you may use `ingress.hosts.extraPaths`
    - Specify the full object of a valid ingress path like shown in [k8s docs](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource) or in [values.yaml: ingress.hosts.extraPaths](values.yaml#L250)

GKE-specific configurations:

- Specify `/*` as the value of `ingress.hosts.paths.path`.
- Comment out `ingress.tls.servicePort` as it is not required.
