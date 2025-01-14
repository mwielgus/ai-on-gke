# TPU Kuberay Webhook Setup Guide

This page contains instructions for how to deploy a mutating admission webhook with Kuberay on TPUs.

### Prerequisites

Preinstall on your computer:
- Docker
- Kubectl
- Terraform
- Helm
- Gcloud

### Installing the GKE Platform

1. If needed, git clone https://github.com/GoogleCloudPlatform/ai-on-gke

2. `cd ai-on-gke/gke-platform`

3. Edit `variables.tf` with your GCP settings.

4. Change the region or zone to one where TPUs are available (see [this link](https://cloud.google.com/tpu/docs/regions-zones) for details. For v4 TPUs (the default type), the region should be set to `us-central2` or `us-central2-b`.

5. Set the following flags (note that TPUs are currently only supported on GKE standard):

### Installing the Webhook

1. `make install-cert-manager`

2. If deploying webhook across multiple namespaces: `make install-reflector`

3. `make all`

4. `make docker-build` - edit Makefile with your own Docker image if necessary

5. `make docker-push`

6. `make deploy`

7. `make deploy-cert`

### Injecting TPU Environment Variables

After deploying the webhook, follow the steps in ray-on-gke/TPU_GUIDE to setup Ray on GKE with TPUs. The webhook will intercept Ray Clusters and Pods created by kuberay and inject environment variables into Pods requesting TPU multi-host resources. Once the Kuberay cluster is deployed, `kubectl describe` the worker pods to verify the `TPU_WORKER_ID`, `TPU_WORKER_HOSTNAMES`, and `TPU_NAME` environment variables have been properly set.

### Limitations

Currently, multi-slice TPUs are not supported by this webhook since we are only deploying a single headless service. However, single-slice, mutli-host TPUs are supported. Additionally,the webhook stores unique `TPU_WORKER_ID`s in memory, and thus will fail to initialize the environment variables correctly if the webhook pod dies or restarts before intercepting all pods. Finally, environment vars are not updated or removed after the initial admission request.