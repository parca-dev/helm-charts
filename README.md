## Helm charts for parca-server and parca-agent

### Usage

```console
helm repo add parca https://parca-dev.github.io/helm-charts/
helm repo update parca
helm install my-parca parca/parca
```

### Contributing
If you added, removed or adjusted helm values, please run [helm-docs](https://github.com/norwoodj/helm-docs) in charts/parca folder to re-generate README