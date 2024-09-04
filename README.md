# Homelab

As a DevOps and Cloud Native Engineer, I need a personal playground to experiment in—without tinkering with client clusters! That’s why I set up my homelab and chronicled the journey on my blog [here](https://alfonsofortunato.com/posts/homelab/).

The core principles for my homelab are:

- Following GitOps best practices
- Simulating real-world scenarios (dev/stage/prod cluster)
- Prioritizing cost-effective hardware choices
- Always experimenting

# Hardware Setup
At the time of writing, the hardware setup is minimal, running just one cluster—affectionately named the **"Gandalf Cluster"**—composed of four Raspberry Pi 4B units, each with 8GB RAM:
- 1 control plane
- 3 workers

Powered via 4 Ethernet PoE and equipped with 64GB microSD storage.
Storage is managed both in-memory and on a Synology NAS with 8TB attached as an iSCSI drive to the cluster.

# Software Setup
Lately, I’ve read good things about Talos OS, and my homelab was the perfect opportunity to try out this new minimal distribution designed for Kubernetes.
All the Raspberry Pis are running Talos OS, customized for my needs. Detailed info on my setup can be found [here](https://alfonsofortunato.com/posts/homelab-talos/).

# IaC and GitOps
FluxCD is my go-to tool for implementing GitOps, allowing me to customize my setup for multiple clusters (currently just one, but more in the future).

The repo structure follows the best practices documented [here](https://fluxcd.io/flux/guides/repository-structure/).

All apps, monitoring tools, infrastructure, etc., are deployed with either Helm or Kustomize builds. These are handled natively by FluxCD via its Kustomize and Helm operators.
Whenever you want to make changes, just push the code to this Git repo, and FluxCD will pull the changes and apply them to the cluster.

# Secrets Handling
I’ve taken a hybrid approach to secrets management. I discovered the beautiful integration of SOPS with FluxCD (explained [here](https://fluxcd.io/flux/guides/mozilla-sops/)), so I decided to give it a try.
However, I wasn’t entirely satisfied for a few reasons:
- The encrypt and decrypt mechanism can be cumbersome in the long run.
- If you lose the private key: bye-bye data.
- Changing a secret value is painful.
- You can’t implement a rotation mechanism.

That’s why I also introduced Azure KeyVault. It costs less than €1/month for 50 secrets, and you can easily change values in a central place without hassle.
Secrets integration is handled via the External Secret Operator. More info [here](https://external-secrets.io/v0.4.3/provider-azure-key-vault/).

In the end:
- SOPS for secrets stored as manifests in the cluster
- Azure KeyVault for everything else
