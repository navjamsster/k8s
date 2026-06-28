helm create myapp
cd myapp
helm lint myapp
helm template myrelease ./myapp
helm install myrelease ./myapp --dry-run=client --debug

helm install myrelease ./myapp


helm install myrelease ./myapp -f values-dev.yaml
helm upgrade myrelease ./myapp --set image.tag=1.2.0