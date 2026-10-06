# Cloud-Native Kubernetes on Oracle Cloud: Deploy and Optimize with OKE

Lab code for the **Oracle AI World 2026** training session **OUT1013**, delivered by Oracle University.

This repository holds the Kubernetes manifests, helper scripts and lab notes used in nine practical labs on Oracle Kubernetes Engine (OKE). Each learner builds two clusters in a dedicated compartment, deploys a multi tier application, pulls images from a private registry, attaches persistent storage and configures autoscaling. The step by step instructions are in the course activity guide; this repository supplies every file the guide asks you to run.

> **Before you start:** the commands assume OCI Cloud Shell with the **public network** enabled (needed for `git clone` and Docker Hub pulls). Your instructor provides your learner ID, region, compartment and any values marked "instructor" below.

## Labs

| Lab | Topic | Cluster | Files in this repository |
|---|---|---|---|
| Start | Prepare Cloud Shell, variables and SSH key | None | `lab.env.example`, `scripts/save-var.sh` |
| 1 | Create a cluster with Quick Create (managed nodes) | managed | `scripts/discover.sh`, `scripts/check-node-ssh-key.sh` |
| 2 | Set up Cloud Shell access to clusters | managed | `scripts/check-context.sh` |
| 3 | Set up local access to clusters | managed | (OCI CLI and kubectl on your laptop) |
| 4 | SSH to a managed node in a private subnet using Cloud Shell | managed | `scripts/lab04-worker-info.sh`, `scripts/check-node-ssh-key.sh`, `scripts/lab04-fix-node-key.sh` |
| 5 | Create a cluster with Quick Create (virtual nodes) | virtual | `manifests/lab05-virtual-nodes/` |
| 6 | Deploy a multi tier application with kubectl | managed | `manifests/lab06-guestbook/` |
| 7 | Pull images from an OCIR repository during deployment | managed | `manifests/lab07-ocir/`, `scripts/lab07-render-frontend.sh` |
| 8 | Provision PVCs on the OCI Block Volume service | managed | `manifests/lab08-block-volume/` |
| 9 | Autoscale node pools and pods with cluster add-ons | managed | `manifests/lab09-autoscaling/`, `scripts/lab09-ca-config.sh`, `scripts/lab09-node-scale-demo.sh` |
| End | Final teardown | both | `scripts/teardown-check.sh` |

## Quick start in Cloud Shell

Clone the repository into the folder name the activity guide uses, so every command in the guide works as written:

```bash
git clone https://github.com/ou-developers/OAIW26-OKE-CLASS.git ~/oaiw26-oke-lab-kit
cd ~/oaiw26-oke-lab-kit
chmod +x scripts/*.sh
cp lab.env.example lab.env
nano lab.env      # set STUDENT_ID, OCI_REGION, OCIR_REGION_KEY, COMPARTMENT_OCID, CA_AUTH_TYPE
source lab.env
```

Load your values automatically in every new Cloud Shell session:

```bash
LINE='[ -f ~/oaiw26-oke-lab-kit/lab.env ] && source ~/oaiw26-oke-lab-kit/lab.env'
grep -qF "$LINE" ~/.bashrc || echo "$LINE" >> ~/.bashrc
```

After Lab 1, Lab 5 and Lab 7, run `scripts/discover.sh` and `source lab.env` again so the cluster, node pool and tenancy values are saved.

## Repository layout

```text
.
├── lab.env.example            # learner variables and naming convention (copy to lab.env)
├── scripts/                   # helper scripts (bash, run in Cloud Shell)
├── manifests/
│   ├── lab05-virtual-nodes/   # vn-web Deployment and Service for virtual nodes
│   ├── lab06-guestbook/       # Guestbook: Redis leader, Redis followers, PHP frontend, LoadBalancer
│   ├── lab07-ocir/            # frontend template that pulls from private OCIR with ocirsecret
│   ├── lab08-block-volume/    # PVC on oci-bv and an nginx pod that mounts it
│   └── lab09-autoscaling/     # php-apache, HPA, bounded load Job, node scale demo, add-on JSON
├── rendered/                  # output of the render scripts (generated, not committed)
└── reference/                 # original 2025 course files and a diff of every adaptation
```

## Helper scripts

| Script | What it does |
|---|---|
| `save-var.sh NAME VALUE` | Sets or updates one value in `lab.env` |
| `discover.sh` | Finds your cluster, node pool and tenancy namespace with the OCI CLI and saves them in `lab.env` |
| `check-context.sh CONTEXT` | Stops if kubectl points at the wrong cluster, then lists its nodes |
| `lab04-worker-info.sh` | Shows worker IPs and the VCN and subnet to choose for Cloud Shell private networking |
| `check-node-ssh-key.sh` | Read only check that your SSH key is on the node pool and on each worker node |
| `lab04-fix-node-key.sh [ip]` | Recovery: sets your key on the node pool and replaces one node (asks for confirmation) |
| `lab07-render-frontend.sh` | Writes the Lab 7 frontend manifest with your OCIR image path |
| `lab09-ca-config.sh` | Writes the Cluster Autoscaler add-on configuration with bounds of current size to current size plus one |
| `lab09-node-scale-demo.sh` | Writes a Deployment sized to need exactly one more node |
| `teardown-check.sh` | Read only list of lab resources still in your compartment |

## Naming used in every lab

| Item | Value |
|---|---|
| Managed node cluster / kubectl context | `<student-id>-oke-managed` / `<student-id>-managed` |
| Virtual node cluster / kubectl context | `<student-id>-oke-virtual` / `<student-id>-virtual` |
| Namespaces | `vn-lab` (virtual cluster); `guestbook`, `storage-lab`, `autoscale-lab` (managed cluster) |
| SSH key pair | `~/.ssh/<student-id>-oke-key` and `.pub` |
| OCIR repository and image pull secret | `oaiw26/<student-id>/guestbook-frontend:v3`, secret `ocirsecret` |

## Container images

| Image | Used in |
|---|---|
| `docker.io/mahidocker2018/guestbook:v3` | Lab 6 frontend; source image copied to OCIR in Lab 7 |
| `registry.k8s.io/redis` (pinned digest) and `us-docker.pkg.dev/google-samples/containers/gke/gb-redis-follower:v2` | Lab 6 Redis tiers |
| `docker.io/library/nginx:1.27` | Labs 5, 8 and 9 |
| `registry.k8s.io/hpa-example` and `docker.io/library/busybox:1.36` | Lab 9 |

## Keep secrets out of this repository

This is a public repository. Never commit `lab.env` (it holds your compartment and cluster OCIDs), auth tokens, API signing keys, SSH private keys or kubeconfig files. The included `.gitignore` excludes these files; check `git status` before every commit.

## Validation

Manifests pass `kubeconform` in strict mode against Kubernetes 1.34, 1.35 and 1.36 schemas and `yamllint`. Scripts pass `shellcheck`. The labs are being rehearsed in OCI; fixes found during rehearsal are applied to both the activity guide and this repository.

## Attribution

The Guestbook and php-apache manifests derive from the Kubernetes documentation examples ([kubernetes/website](https://github.com/kubernetes/website), content licensed CC BY 4.0), which credit Google's GKE Guestbook tutorial. Original course files from the 2025 edition ([OAIW25-OKE-CLASS](https://github.com/ou-developers/OAIW25-OKE-CLASS), commit `6dcf757`) are kept unchanged in `reference/`, with every adaptation listed in `reference/changes-from-original.diff`.

## References

* [Oracle Kubernetes Engine documentation](https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengoverview.htm)
* [Setting up cluster access](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengdownloadkubeconfigfile.htm)
* [Cloud Shell networking](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellintro_topic-Cloud_Shell_Networking.htm)
* [OKE cluster add-ons](https://docs.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengintroducingclusteraddons.htm)
