# Setting up the cluster
First, you will need a few CLI tools:
- `talosctl`
- `cilium-cli`
- `flux`
- `kubectl`

## Talos
### Creating a talos config
TODO

### Applying the config
We currently have 1 control plane:
- Mega

And no workers currently, as we run services on the controlplanes.

Use this command to apply the control plane config
`talosctl apply-config -n <ip> -e <ip> --file controlplane.yaml`
Use this command to apply the worker config
`talosctl apply-config -n <ip> -e <ip> --file controlplane.yaml`

After that, bootstrap every node
`talosctl bootstrap -n <ip> -e <ip>`

### Generating a kubeconfig
Use `talosctl gen config cube https://10.4.0.10:6443` to generate a kubeconfig in your local dir.  
You should either:
- Set the `KUBECONFIG` environment variable to the absolute path of the config to make kubernetes tools use that kubeconfig
- Move it to `~/.config/kube/config` to make it the global default for kubernetes tools.

## Cilium
Run this command to install it
```sh
cilium install \
    --set ipam.mode=kubernetes \
    --set kubeProxyReplacement=true \
    --set securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
    --set securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}" \
    --set cgroup.autoMount.enabled=false \
    --set cgroup.hostRoot=/sys/fs/cgroup \
    --set k8sServiceHost=localhost \
    --set k8sServicePort=7445
```
## Flux
Create a deploy key to this repo and use this command flux to the repo
`flux bootstrap git --url=ssh://git@github.com/kubeside/k8s --branch=master --private-key-file=flux_ed25519 --path clusters/cube`
After a while the system services and apps will start running
