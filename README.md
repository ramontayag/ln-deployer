# LightningNetwork Deployer

Kubernetes deployment files for Bitcoin LightningNetwork. It contains both testnet and mainnet files. These are meant to be used as a guide, and not to be deployed from directly.

## Security

When deploying production, you should be using your own images and not trusting the images that other people have built. This does not cover that.

## Setup

Before you start here, you should be able to talk to your Kubernetes cluster. `kubectl get nodes` should return the nodes you have.

All `$CONTEXT` variables below is the name of your Kube context. On Google Cloud, it looks like `gke_project-name_us-central1-a_cluster-name`. You can just `export CONTEXT=my-context` before proceeding to avoid having to copy and paste it each time.

Download and install libraries for this deployer:

```
git clone ln-deployer
cd ln-deployer
bundle
```

## bitcoind

First, `cd` into the bitcoind deployment files:

```
cd bitcoind
```

Create the namespace:

```
kubectl create namespace ln-bitcoind-testnet
```

Setup the encryption secrets:

```
ejson keygen
kubectl create secret generic ejson-keys --from-literal=public_key_above=private_key_above --namespace=ln-bitcoind-testnet
```

Copy the secret encryption file:

```
cp config/deploy/testnet/secrets.ejson{.sample,}
```

With a text editor, change the value of `_public_key` of `secrets.ejson` to the public key generated above. Change `rpcpassword` to a password you want.

Encrypt it:

```
ejson encrypt config/deploy/testnet/secrets.ejson
```

Deploy bitcoind:

```
ENVIRONMENT=testnet kubernetes-deploy ln-bitcoind-testnet $CONTEXT
```

Once that has synced, which may take up to a few days, deploy lnd.

## LND

`cd` into the `lnd` directory:

```
cd ../lnd
```

Create the namespace:

```
kubectl create namespace ln-lnd-testnet
```

Setup the encryption secrets:

```
ejson keygen
kubectl create secret generic ejson-keys --from-literal=public_key_above=private_key_above --namespace=ln-lnd-testnet
```

Copy the secret encryption file:

```
cp config/deploy/testnet/secrets.ejson{.sample,}
```

With a text editor, change the value of `_public_key` of `secrets.ejson` to the public key generated above. Change `rpcpassword` to match the bitcoind password above.

Encrypt it:

```
ejson encrypt config/deploy/testnet/secrets.ejson
```

Deploy lnd:

```
ENVIRONMENT=testnet kubernetes-deploy ln-lnd-testnet $CONTEXT
```

Create the wallet:

```
kubectl exec `kubectl get pods -n ln-lnd-testnet -l app=lnd | tail -1 | awk '{print $1}'` -n ln-lnd-testnet -it -- bash
lncli create # enter your password
```
