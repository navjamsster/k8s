https://killercoda.com/cka-mock-practice/scenario/fix-rbac

Namespace: qa-tools
├── Deployment: pod-explorer
│   └── Uses ServiceAccount: sa-explorer
├── ServiceAccount: sa-explorer
├── Roles (predefined by security team)
│   ├── config-reader (ConfigMaps only)
│   ├── secret-reader (Secrets only)
│   ├── pod-reader ← Selected ✓
│   └── deployment-viewer (Deployments only)
└── RoleBinding: explorer-rolebinding
    ├── Binds: pod-reader → sa-explorer
    └── Grants: get, list, watch on pods


https://killercoda.com/cka-mock-practice/scenario/GitLab-CICD-RBAC