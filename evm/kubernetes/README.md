# Chaguar EVM on Kubernetes

This is the Kubernetes version of the [EVM sample](../). It deploys a
minimal Chaguar network with the EVM gateway in a local
[kind](https://kind.sigs.k8s.io/) cluster, using the `lfdt.chaguar` Ansible
collection.

What you get:

- 4 orderer groups (router, consenter, assembler and batcher each)
- 1 committer (validator, verifier, coordinator, sidecar, query service) with PostgreSQL
- the EVM gateway, serving Ethereum JSON-RPC on `localhost:8545`, chain ID `4011` (`0xfab`)
- optionally, the Chaguar block explorer (see [Add the Chaguar block explorer](#add-the-chaguar-block-explorer))

Crypto material comes from `cryptogen`. There is no Fabric CA, monitoring or
load generator.

> **Note:** The Blockscout EVM explorer from the [EVM sample](../) is not
> available in this setup. Only the Chaguar block explorer, which shows raw
> Chaguar blocks rather than EVM transactions, can be added.

**Prerequisites:** Docker, `kind`, `kubectl`, [`uv`](https://docs.astral.sh/uv/)
or Ansible, and
[Foundry](https://book.getfoundry.sh/getting-started/installation) for the
token demo.

See the [docs](https://chaguar.github.io/ansible-collection/) for more info about
this deployment and how to configure it.

## 1. Install Ansible and the collection

```shell
uv tool install ansible-core --with kubernetes
ansible-galaxy collection install --upgrade 'lfdt.chaguar:>=0.12.3'
```

The second command also installs the collections `lfdt.chaguar` depends on
(`kubernetes.core`, `community.docker`, and others).

## 2. Go to this folder

From the root of this repository:

```shell
cd evm/kubernetes
```

Run all following commands from here. The folder holds three files:

- [`kind-config.yaml`](kind-config.yaml): a single-node kind cluster that
  publishes the NodePorts the control node and EVM clients use on `localhost`.
- [`inventory.yaml`](inventory.yaml): the network topology.
- [`ansible.cfg`](ansible.cfg): points Ansible at the inventory, starts more
  workloads in parallel and keeps the output short. Ansible reads it from the
  directory you run it in.

The network tooling runs in containers that bind-mount this folder. If Docker
runs in a VM (as on macOS), the VM must have access to this folder.

## 3. Create the kind cluster

```shell
kind create cluster --name chaguar --config kind-config.yaml --wait 120s
kubectl get nodes   # current context is now kind-chaguar
```

## 4. Deploy

This generates the crypto material and the genesis block, renders the
configs, starts all workloads, and creates the `basic` namespace. It takes
about 5 minutes. Generated files go to `./out`.

```shell
ansible-playbook lfdt.chaguar.deploy
```

Check the pods:

```shell
kubectl get pods
```

All pods should be `Running`, except `committer-validator-init-db-*`, which
ends as `Completed`.

## 5. Use the EVM gateway

```shell
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"eth_chainId","params":[]}' http://localhost:8545
# {"jsonrpc":"2.0","id":1,"result":"0xfab"}
```

The gateway listens on the standard Ethereum RPC port, so the token demo of
the [EVM sample](../) works unchanged:

```shell
cd ..
make deploy NAME="Digital Euro" SYMBOL=EURX
make transfer TO=0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 AMOUNT=500
cd kubernetes
```

The links they print point to Blockscout, which this setup doesn't run.

Foundry, Hardhat and MetaMask work the same way: use `http://localhost:8545`
and chain ID `4011`. See
[Chaguar/evm](https://github.com/Chaguar/evm#build-your-own-contracts) for
caveats.

## 6. Stop and clean up

To remove the network but keep the cluster (redeploy with step 4):

```shell
ansible-playbook lfdt.chaguar.teardown
```

To remove everything:

```shell
kind delete cluster --name chaguar
rm -rf out
```

## Add the Chaguar block explorer

The block explorer shows the raw Chaguar blocks and transactions behind each
EVM transaction. To add it:

1. Uncomment the `chaguar_block_explorer` group in `inventory.yaml`.
2. Run the deploy again (step 4). You don't need to recreate the cluster,
   because `kind-config.yaml` already publishes port `30690`.
3. Open <http://localhost:30690>.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `cryptogen: error: open /tmp/cryptogen-artifacts/crypto-config.yaml` | This folder is not shared with the Docker VM. Add it to the VM's shared paths, or move the repository to a folder that is shared, such as your home directory. |
| `Failed to import the required Python library (kubernetes)` | Reinstall Ansible with `--with kubernetes` (step 1). If you use another Ansible install, add the `kubernetes` Python package to it. |
| `the playbook: lfdt.chaguar.deploy could not be found` | Your `lfdt.chaguar` is too old. Run step 1 again. |
| `No inventory was parsed, only implicit localhost is available` | Run `ansible-playbook` from this folder, so Ansible finds `ansible.cfg`. |
| Port already allocated when creating the cluster | Another process or kind cluster uses one of the ports in `kind-config.yaml` (8545 and 30050–30690). Free it or delete the other cluster. |
