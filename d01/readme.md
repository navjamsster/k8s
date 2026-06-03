kubectl get pods -l uses=not-useful -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priority,CLASS:.spec.priorityClassName" --sort-by='.spec.priority'

This command lists pods that have the label uses=not-useful. The -l option filters by that label selector so only matching pods are shown.

The -o custom-columns output format defines three columns: NAME from .metadata.name, PRIORITY from .spec.priority, and CLASS from .spec.priorityClassName. This helps compare a pod's numeric priority value with its priority class name.

The --sort-by='.spec.priority' option sorts the displayed pods by the numeric priority field. Note that if a pod lacks a numeric .spec.priority (for example if only priorityClassName is set), sorting behavior will reflect that absence.