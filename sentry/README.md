# Install

## Add repo

```
helm repo add sentry https://sentry-kubernetes.github.io/charts
```

## Without overrides

```
helm install sentry sentry/sentry
```

## With your own vaLues file

```
helm install sentry sentry/sentry -f values.yaml
```

## Upgrading from 11.x.x version of this Chart to 12.0.0

Redis chart was upgraded to newer version. If you are using external redis, you don't need to do anything.

Otherwise, when upgrading to chart version 12.x.x from 11.x.x you need to either run `helm upgrade` with `--force` flag, or prior to upgrade delete statefulsets for redis master and redis slave. Then run upgrade and it will roll out new statefulsets. Your master redis data will not be lost (PVC is not deleted when you delete statefulset). Your redis slave will now be named redis replica and you can delete PVCs that were used by redis slave after the upgrade.

## Upgrading from 10.x.x version of this Chart to 11.0.0

If you were using clickhouse tabix externally, we disabled it per default.

### Upgrading from deprecated 9.0 -> 10.0 Chart

As this chart runs in helm 3 and also tries its best to follow on from the original Sentry chart. There are some steps that needs to be taken in order to correctly upgrade.

From the previous upgrade, make sure to get the following from your previous installation:

- Redis Password (If Redis auth was enabled)
- Postgresql Password
  Both should be in the `secrets` of your original 9.0 release. Make a note of both of these values.

#### Upgrade Steps

Due to an issue where transferring from Helm 2 to 3. Statefulsets that use the following: `heritage: {{ .Release.Service }}` in the metadata field will error out with a `Forbidden` error during the upgrade. The only workaround is to delete the existing statefulsets (Don't worry, PVC will be retained):

> kubectl delete --all sts -n <Sentry Namespace>

Once the statefulsets are deleted. Next steps is to convert the helm release from version 2 to 3 using the helm 3 plugin:

> helm3 2to3 convert <Sentry Release Name>

Finally, it's just a case of upgrading and ensuring the correct params are used:

If Redis auth enabled:

> helm upgrade -n <Sentry namespace> <Sentry Release> . --set redis.usePassword=true --set redis.password=<Redis Password>

If Redis auth is disabled:

> helm upgrade -n <Sentry namespace> <Sentry Release> .

## Configuration

The following table lists the configurable parameters of the Sentry chart and their default values.

Note: this table is incomplete, so have a look at the values.yaml in case you miss something

| Parameter                                     | Description                                                                                                                                                         | Default                        |
| :-------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------- |
| `user.create`                                 | if `true`, creates a default admin user defined from `email` and `password`                                                                                         | `true`                         |
| `user.email`                                  | Admin user email                                                                                                                                                    | `admin@sentry.local`           |
| `user.password`                               | Admin user password                                                                                                                                                 | `aaaa`                         |
| `ingress.enabled`                             | Enabling Ingress                                                                                                                                                    | `false`                        |
| `ingress.regexPathStyle`                      | Allows setting the style the regex paths are rendered in the ingress for the ingress controller in use. Possible values are `nginx`, `aws-alb`, `gke` and `traefik` | `nginx`                        |
| `nginx.enabled`                               | Enabling NGINX                                                                                                                                                      | `true`                         |
| `metrics.enabled`                             | if `true`, enable Prometheus metrics                                                                                                                                | `false`                        |
| `metrics.image.repository`                    | Metrics exporter image repository                                                                                                                                   | `prom/statsd-exporter`         |
| `metrics.image.tag`                           | Metrics exporter image tag                                                                                                                                          | `v0.10.5`                      |
| `metrics.image.PullPolicy`                    | Metrics exporter image pull policy                                                                                                                                  | `IfNotPresent`                 |
| `metrics.nodeSelector`                        | Node labels for metrics pod assignment                                                                                                                              | `{}`                           |
| `metrics.tolerations`                         | Toleration labels for metrics pod assignment                                                                                                                        | `[]`                           |
| `metrics.affinity`                            | Affinity settings for metrics                                                                                                                                       | `{}`                           |
| `metrics.resources`                           | Metrics resource requests/limit                                                                                                                                     | `{}`                           |
| `metrics.service.annotations`                 | annotations for Prometheus metrics service                                                                                                                          | `{}`                           |
| `metrics.service.clusterIP`                   | cluster IP address to assign to service (set to `"-"` to pass an empty value)                                                                                       | `nil`                          |
| `metrics.service.omitClusterIP`               | (Deprecated) To omit the `clusterIP` from the metrics service                                                                                                       | `false`                        |
| `metrics.service.externalIPs`                 | Prometheus metrics service external IP addresses                                                                                                                    | `[]`                           |
| `metrics.service.additionalLabels`            | labels for metrics service                                                                                                                                          | `{}`                           |
| `metrics.service.loadBalancerIP`              | IP address to assign to load balancer (if supported)                                                                                                                | `""`                           |
| `metrics.service.loadBalancerSourceRanges`    | list of IP CIDRs allowed access to load balancer (if supported)                                                                                                     | `[]`                           |
| `metrics.service.servicePort`                 | Prometheus metrics service port                                                                                                                                     | `9913`                         |
| `metrics.service.type`                        | type of Prometheus metrics service to create                                                                                                                        | `ClusterIP`                    |
| `metrics.serviceMonitor.enabled`              | Set this to `true` to create ServiceMonitor for Prometheus operator                                                                                                 | `false`                        |
| `metrics.serviceMonitor.additionalLabels`     | Additional labels that can be used so ServiceMonitor will be discovered by Prometheus                                                                               | `{}`                           |
| `metrics.serviceMonitor.honorLabels`          | honorLabels chooses the metric's labels on collisions with target labels.                                                                                           | `false`                        |
| `metrics.serviceMonitor.namespace`            | namespace where servicemonitor resource should be created                                                                                                           | `the same namespace as sentry` |
| `metrics.serviceMonitor.scrapeInterval`       | interval between Prometheus scraping                                                                                                                                | `30s`                          |
| `serviceAccount.annotations`                  | Additional Service Account annotations.                                                                                                                             | `{}`                           |
| `serviceAccount.enabled`                      | If `true`, a custom Service Account will be used.                                                                                                                   | `false`                        |
| `serviceAccount.name`                         | The base name of the ServiceAccount to use. Will be appended with e.g. `snuba` or `web` for the pods accordingly.                                                   | `"sentry"`                     |
| `serviceAccount.automountServiceAccountToken` | Automount API credentials for a Service Account.                                                                                                                    | `true`                         |
| `system.secretKey`                            | secret key for the session cookie ([documentation](https://develop.sentry.dev/config/#general))                                                                     | `nil`                          |
| `sentry.features.vstsLimitedScopes`           | Disables the azdo-integrations with limited scopes that is the cause of so much pain                                                                                | `true`                         |
| `sentry.web.customCA.secretName`              | Allows mounting a custom CA secret                                                                                                                                  | `nil`                          |
| `sentry.web.customCA.item`                    | Key of CA cert object within the secret                                                                                                                             | `ca.crt`                       |
| `symbolicator.api.enabled`                    | Enable Symbolicator                                                                                                                                                 | `false`                        |
| `symbolicator.api.config`                     | Config file for Symbolicator, see [its docs](https://getsentry.github.io/symbolicator/#configuration)                                                               | see values.yaml                |

## NGINX and/or Ingress

By default, NGINX is enabled to allow sending the incoming requests to [Sentry Relay](https://getsentry.github.io/relay/) or the Django backend depending on the path. When Sentry is meant to be exposed outside of the Kubernetes cluster, it is recommended to disable NGINX and let the Ingress do the same. It's recommended to go with the go to Ingress Controller, [NGINX Ingress](https://kubernetes.github.io/ingress-nginx/) but others should work as well.

## Sentry secret key

For your security, the [`system.secret-key`](https://develop.sentry.dev/config/#general) is generated for you on the first installation. Another one will be regenerated on each upgrade invalidating all the current sessions unless it's been provided. The value is stored in the `sentry-sentry` configmap.

```
helm upgrade ... --set system.secretKey=xx
```

## Symbolicator

For getting native stacktraces and minidumps symbolicated with debug symbols (e.g. iOS/Android), you need to enable Symbolicator via

```yaml
symbolicator:
  enabled: true
```

However, you also need to share the data between sentry-worker and sentry-web. This can be done in different ways:

- Using Cloud Storage like GCP GCS or AWS S3, see `filestore.backend` in `values.yaml`
- Using a filesystem like

```yaml
filestore:
  filesystem:
    persistence:
      persistentWorkers: true
      # storageClass: 'efs-storage' # see note below
```

Note: If you need to run or cannot avoid running sentry-worker and sentry-web on different cluster nodes, you need to set `filestore.filesystem.persistence.accessMode: ReadWriteMany` or might get problems. HOWEVER, [not all volume drivers support it](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes), like AWS EBS or GCP disks.
So you would want to create and use a `StorageClass` with a supported volume driver like [AWS EFS](https://github.com/kubernetes-sigs/aws-efs-csi-driver)

Its also important having `connect_to_reserved_ips: true` in the symbolicator config file, which this Chart defaults to.

# Usage with Terraform + AWS

`./templates/sentry_values.yaml` file

```yaml
prefix: ${module_prefix}

user:
  create: true
  email: ${sentry_email}
  password: ${sentry_password}

nginx:
  enabled: false

rabbitmq:
  enabled: false

sentry:
  web:
    service:
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /_health/
        alb.ingress.kubernetes.io/healthcheck-port: traffic-port

relay:
  service:
    annotations:
      alb.ingress.kubernetes.io/healthcheck-path: /api/relay/healthcheck/ready/
      alb.ingress.kubernetes.io/healthcheck-port: traffic-port

postgresql:
  enabled: true
  nameOverride: sentry-postgresql
  postgresqlUsername: postgres
  postgresqlPassword: ${postgres_password}
  postgresqlDatabase: sentry
  replication:
    enabled: false

ingress:
  enabled: true
  hostname: ${sentry_dns_name}
  regexPathStyle: aws-alb
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/tags: ${tags}
    alb.ingress.kubernetes.io/inbound-cidrs: ${allowed_cidr_blocks_str}
    alb.ingress.kubernetes.io/subnets: ${public_subnet_ids_str}
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-redirect: "443"
    alb.ingress.kubernetes.io/certificate-arn: ${subdomain_cert_arn}
    external-dns.alpha.kubernetes.io/hostname: ${sentry_dns_name}
```

`./helm.tf` file

```terraform
resource "helm_release" "sentry" {
  name  = "sentry"
  chart = "${path.module}/helm_sentry/"
  repository = "https://sentry-kubernetes.github.io/charts"
  version    = "13.0.0"
  timeout           = 600
  wait              = false
  dependency_update = true

  values = [
    templatefile(
      "${path.module}/templates/sentry_values.yaml",
      {
        module_prefix   = "${var.module_prefix}",
        sentry_email    = "${var.sentry_email}",
        sentry_password = "${var.sentry_password}",

        sentry_dns_name         = "${local.sentry_dns_name}",
        subdomain_cert_arn      = "${var.subdomain_cert_arn}",
        allowed_cidr_blocks_str = "${join(",", var.allowed_cidr_blocks)}",
        private_subnet_ids_str  = "${join(",", var.private_subnet_ids)}",
        public_subnet_ids_str   = "${join(",", var.public_subnet_ids)}",
        tags                    = "environment=${var.env}"
        # postgres_db_host        = "${module.sentry_rds_pg.this_rds_cluster_endpoint}",
        # postgres_db_name        = "${local.db_name}",
        postgres_username = "${local.db_user}",
        postgres_password = "${local.db_pass}",
      }
    )
  ]

  depends_on = [
    helm_release.lb_controller,
    helm_release.external_dns,
  ]
}
```

### Notes

1. Ensure the control plane and node security groups are appropriately configured as documented [here](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html#control-plane-worker-node-sgs).
2. Annotations for ingress are as mentioned [here](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/guide/ingress/annotations/)
3. `healthcheck-path` and `healthcheck-port` annotations can be setup per target group using the alb annotations in the corresponding services as mentioned [here](https://github.com/kubernetes-sigs/aws-load-balancer-controller/issues/1056#issuecomment-551585078). For example, here we have:

```yaml
sentry:
  web:
    service:
      annotations:
        alb.ingress.kubernetes.io/healthcheck-path: /_health/
        alb.ingress.kubernetes.io/healthcheck-port: traffic-port

relay:
  service:
    annotations:
      alb.ingress.kubernetes.io/healthcheck-path: /api/relay/healthcheck/ready/
      alb.ingress.kubernetes.io/healthcheck-port: traffic-port
```

Which are load balancer annotations specified in the service configuration for the load balancer to pick while creating the target groups.

NOTE: AWS ALB Controller's Service annotations don't apply here as we want the `aws-load-balancer-controller` to pick-up the services and apply the appropriate healthcheck-path per service and not create a load balancer for the service itself. The service annotations will only apply when you want the service to be load balanced.
