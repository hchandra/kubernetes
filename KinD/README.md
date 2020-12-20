https://kind.sigs.k8s.io/docs/user/quick-start/


kubectl cluster-info --context kind-kind
kind create cluster --config kind-multi-node-config.yaml
kind create cluster --config kind-ha-multi-node-config.yaml

