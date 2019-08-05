##Ramped
kubectl apply -f app-v1.yaml

watch -n 1 kubectl get pod
while sleep 0.1; do curl 13.68.130.214:5678; done

kubectl apply -f app-v2.yaml

kubectl rollout undo deploy my-app
kubectl delete deployment fruit-ramped-v1

##Canary
kubectl apply -f app-v1.yaml

watch -n 1 kubectl get pod
while sleep 0.1; do curl 13.68.138.168:5678; done

kubectl apply -f app-v2.yaml

kubectl scale --replicas=5 deploy fruit-canary-v2
kubectl delete deploy fruit-canary-v1

##Blue-Green
kubectl apply -f app-v1.yaml

watch -n 1 kubectl get pod
while sleep 0.1; do curl 13.92.158.13:5678; done

kubectl apply -f app-v2.yaml
kubectl rollout status deploy fruit-bg-v2 -w

kubectl patch service fruit-service-bg -p '{"spec":{"selector":{"version":"v2.0.0"}}}'
kubectl patch service fruit-service-bg -p '{"spec":{"selector":{"version":"v1.0.0"}}}'

kubectl delete deploy fruit-bg-v1
