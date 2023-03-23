A unica diferença entre o deployment e o replicaset é o versionamento

kubectl rollout history deployment nginx-deployment

kubectl annotate deployment nginx-deployment kubernetes.io/change-cause="Definindo a nova imagem"

kubectl rollout undo deployment nginx-deployment --to-revision=2

kubectl exec -it pod-volume --container nginx-container -- bash

