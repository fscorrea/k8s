# Estrategias de despliegue

La elección del procedimiento de implementación adecuado depende de las necesidades, enumeramos a continuación algunas de las posibles estrategias para adoptar:

* **Recreate:** terminar la versión anterior y lanzar la nueva.
* **Ramped:** lanzar una nueva versión en una actualización de actualización, uno tras otro.
* **Blue /Green:** publique una nueva versión junto con la versión anterior y luego cambie el tráfico.
* **Canary:** publique una nueva versión para un subconjunto de usuarios, luego proceda a un despliegue completo.

## Ramped

Una implementación en rampa actualiza los pods de forma continua, se crea un ReplicaSet secundario con la nueva versión de la aplicación, luego se reduce el número de réplicas de la versión anterior y se aumenta la nueva versión hasta que se alcanza el número correcto de réplicas.

    kubectl apply -f app-v1.yaml
    watch -n 1 kubectl get pod
    while sleep 0.1; do curl 13.68.130.214:5678; done
    kubectl apply -f app-v2.yaml
    kubectl delete app-v1.yaml
    kubectl delete app-v2.yaml

## Canary

Una implementación **canaria** consiste en enrutar un subconjunto de usuarios a una nueva funcionalidad. En Kubernetes, se puede realizar una implementación canaria utilizando dos implementaciones con etiquetas de pod comunes. Se lanza una réplica de la nueva versión junto con la versión anterior. Luego, después de un tiempo y si no se detecta ningún error, aumente el número de réplicas de la nueva versión y elimine la implementación anterior.

    kubectl apply -f app-v1.yaml
    watch -n 1 kubectl get pod
    while sleep 0.1; do curl 13.68.138.168:5678; done
    kubectl apply -f app-v2.yaml
    kubectl scale --replicas=5 deploy fruit-canary-v2
    kubectl delete app-v1.yaml
    kubectl delete app-v2.yaml


## Blue-Green

Una implementación azul / verde difiere de una implementación en rampa porque la versión "verde" de la aplicación se implementa junto con la versión "azul". Después de probar que la nueva versión cumple con los requisitos, actualizamos el objeto del Servicio de Kubernetes que desempeña el papel de equilibrador de carga para enviar tráfico a la nueva versión reemplazando la etiqueta de versión en el campo selector.

    kubectl apply -f app-v1.yaml
    watch -n 1 kubectl get pod
    while sleep 0.1; do curl 13.92.158.13:5678; done
    kubectl apply -f app-v2.yaml
    kubectl patch service fruit-service-bg -p '{"spec":{"selector":{"version":"v2.0.0"}}}'
    kubectl patch service fruit-service-bg -p '{"spec":{"selector":{"version":"v1.0.0"}}}'
    kubectl delete app-v1.yaml
    kubectl delete app-v2.yaml
