# {{ managed-k8s-name }} tutorials

You can use {{ managed-k8s-name }} to deploy, scale, and manage your containerized applications in the {{ yandex-cloud }} infrastructure.

## Creating and setting up a project {#creating-project}

* [{#T}](new-kubernetes-project.md)
* [{#T}](k8s-cluster-with-no-internet.md)
* [{#T}](kubernetes-terraform-provider.md)
* [{#T}](running-pod-gpu.md)
* [{#T}](driverless-gpu.md)
* [{#T}](time-slicing-gpu.md)
* [{#T}](migration-to-an-availability-zone.md)
* [{#T}](kms-k8s.md)
* [{#T}](k8s-cluster-api-provider-yandex.md)

### Creating a project using {{ TF }} {#terraform}

* [{#T}](kubernetes-terraform-provider.md)
* [{#T}](terraform-modules.md)

## Setting up and testing scaling {#scaling}

* [{#T}](autoscaling.md)
* [{#T}](vpa-autoscaling.md)
* [{#T}](metrics-server.md)
* [{#T}](load-testing-grpc-autoscaling.md)

## Installing NGINX {#nginx}

* [{#T}](ingress-cert-manager.md)
* [{#T}](nginx-ingress-certificate-manager.md)


For how to install an NGINX Ingress Controller with the help of {{ marketplace-full-name }}, see [this guide](../operations/applications/ingress-nginx.md).


## {{ container-registry-full-name }} usage {#container-registry}

* [{#T}](container-registry.md)
* [{#T}](sign-cr-with-cosign.md)
* [{#T}](image-storage.md)

## {{ mkf-name }} usage {#kafka}

[{#T}](deploy-kafka-ui.md)

## Continuous integration with {{ GL }} {#gitlab}

* [{#T}](gitlab-containers.md)
* [{#T}](cr-scanner-with-k8s-and-gitlab.md)
* [{#T}](ci-cd-serverless.md)

## Working with DNS {#dns}

* [{#T}](custom-dns.md)
* [{#T}](dns-autoscaler.md)
* [{#T}](node-local-dns.md)
* [{#T}](dnschallenge.md)
* [{#T}](cert-manager-webhook.md)

## Backups {#backup}

* [{#T}](kubernetes-backup.md)
* [{#T}](pvc-snapshot-restore.md)

## Monitoring {#monitoring}

* [{#T}](prometheus-grafana-monitoring.md)
* [{#T}](k8s-fluent-bit-logging.md)
* [{#T}](filebeat-oss-monitoring.md)

## Using {{ marketplace-full-name }} products {#marketplace-tutorials}

* [{#T}](marketplace/argo-cd.md)
* [{#T}](marketplace/crossplane.md)
* [{#T}](kubernetes-lockbox-secrets.md)
* [{#T}](fluent-bit-logging.md)
* [{#T}](marketplace/gateway-api.md)
* [{#T}](alb-ingress-controller.md)
* [{#T}](alb-ingress-controller-log-options.md)
* [{#T}](custom-health-checks.md)
* [{#T}](alb-ingress-with-sws-profile.md)
* [{#T}](marketplace/jaeger-over-ydb.md)
* [{#T}](marketplace/kyverno.md)
* [{#T}](marketplace/metrics-provider.md)
* [{#T}](marketplace/thumbor.md)
* [{#T}](marketplace/istio.md)
* [{#T}](marketplace/hashicorp-vault.md)
