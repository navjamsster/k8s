kubectl create configmap game-config --from-file=./configmap
kubectl describe configmaps game-config
kubectl delete configmaps game-config
kubectl get configmaps game-config -o yaml


kubectl create configmap game-config-2 --from-file=configure-pod-container/configmap/game.properties
kubectl describe configmaps game-config-2


kubectl create configmap game-config-env-file \
       --from-env-file=configure-pod-container/configmap/game-env-file.propertie

--from-file
--from-env-file
--from-literal


You can't use ConfigMaps for static pods, because the kubelet does not support this.
