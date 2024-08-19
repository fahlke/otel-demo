# flux-kind

## Prerequisites

Install all required tools with one command. Check out the `Brewfile` to see what will be installed/upgraded.

```
brew bundle install
```

## Stop collecting my data...

```
ctlptl analytics opt out
skaffold config set --global collect-metrics false
```

## Start Kind Cluster

```
podman machine init \
  --cpus 8 --memory 24576 --disk-size 100 \
  --timezone 'Etc/UTC' --rootful && \
podman machine start && \
export DOCKER_HOST="unix://$HOME/.local/share/containers/podman/machine/podman.sock"
```

## Install services

```
pushd 01-kind-cluster >/dev/null
  ctlptl apply -f ctlptl.yaml
  skaffold run
popd

kubectl apply -k 02-services-infra
```

## Cleanup

```
podman machine stop && \
podman machine rm --save-image --force
```

## Upgrading Flux

```
pushd kind-cluster >/dev/null
  flux install \
    --components 'source-controller,kustomize-controller,helm-controller,notification-controller' \
    --components-extra 'image-reflector-controller,image-automation-controller' \
    --export > k8s/base/flux/gotk-components.yaml
  skaffold run
popd >/dev/null
```

## Optimize your pod resources with KRR

```
kubectl port-forward -n prometheus pod/prometheus-prometheus-kube-prometheus-prometheus-0 9090
krr simple -n cert-manager -p http://127.0.0.1:9090
```
