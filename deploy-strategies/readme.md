# Estrategias de despliegue

La elección del procedimiento de implementación adecuado depende de las necesidades, enumeramos a continuación algunas de las posibles estrategias para adoptar:

* **Recreate:** terminar la versión anterior y lanzar la nueva.
* **Ramped:** lanzar una nueva versión en una actualización de actualización, uno tras otro.
* **Blue /Green:** publique una nueva versión junto con la versión anterior y luego cambie el tráfico.
* **Canary:** publique una nueva versión para un subconjunto de usuarios, luego proceda a un despliegue completo.

_**Importante**: Para todos los ejemplos de ahora en adelante, debemos de abrir 3 consolas de comandos, con el próposito de visualizar los cambios en el k8s de los componentes desplegados._

## Ramped

Una implementación en rampa actualiza los pods de forma continua, se crea un ReplicaSet secundario con la nueva versión de la aplicación, luego se reduce el número de réplicas de la versión anterior y se aumenta la nueva versión hasta que se alcanza el número correcto de réplicas.

En la primera consola vamos a realizar los despliegues de nuestras aplicaciones, para ello ejecutaremos el siguientes comando.

    kubectl apply -f app-v1.yaml

En la segunda consola observaremos como el cluster de k8s realiza el despliegue de las aplicaciones:

    watch -n 1 kubectl get pod
    
En la tercera consola observaremos la información emitada por la aplicación a través del endpoint expuesta por ella.

    while sleep 0.1; do curl 40.70.77.149:5678; done

**Nota:** La dirección ip corresponde al servicio creado en el despliegue.

Por ultimo, en la primera consola desplegamos la versión 2.0.0 de la aplicación y en las otras dos consolas evidenciaremos como se construye la nueva versión de la aplicación y como esta interactua con la versión 1.0.0

    kubectl apply -f app-v2.yaml
    
Después de evidenciar esta estrategia de despliegue, se recomienda eliminar todos los componentes creados en el cluster, para ello ejecutar los siguientes comandos.

    kubectl delete app-v1.yaml
    kubectl delete app-v2.yaml

## Canary

Una implementación **canaria** consiste en enrutar un subconjunto de usuarios a una nueva funcionalidad. En Kubernetes, se puede realizar una implementación canaria utilizando dos implementaciones con etiquetas de pod comunes. Se lanza una réplica de la nueva versión junto con la versión anterior. Luego, después de un tiempo y si no se detecta ningún error, aumente el número de réplicas de la nueva versión y elimine la implementación anterior.

En la primera consola vamos a realizar los despliegues de nuestras aplicaciones, para ello ejecutaremos el siguientes comando.

    kubectl apply -f app-v1.yaml

En la segunda consola observaremos como el cluster de k8s realiza el despliegue de las aplicaciones:

    watch -n 1 kubectl get pod
    
En la tercera consola observaremos la información emitada por la aplicación a través del endpoint expuesta por ella.

    while sleep 0.1; do curl {ip}; done

**Nota:** La dirección ip corresponde al servicio creado en el despliegue.

Por ultimo, en la primera consola desplegamos la versión 2.0.0 de la aplicación y en las otras dos consolas se evidenciará como se construye la nueva versión de la aplicación y como esta interactua con la versión 1.0.0

    kubectl apply -f app-v2.yaml

Para aumentar el nivel de solicitudes a la nueva versión, con el siguiente comando se aumnetará el número de instancias de la versión 2.0.0

    kubectl scale --replicas=5 deploy fruit-canary-v2
    
Después de evidenciar esta estrategia de despliegue, se recomienda eliminar todos los componentes creados en el cluster, para ello ejecutar los siguientes comandos.

    kubectl delete app-v1.yaml
    kubectl delete app-v2.yaml

## Blue-Green

Una implementación azul / verde difiere de una implementación en rampa porque la versión "verde" de la aplicación se implementa junto con la versión "azul". Después de probar que la nueva versión cumple con los requisitos, actualizamos el objeto del Servicio de Kubernetes que desempeña el papel de equilibrador de carga para enviar tráfico a la nueva versión reemplazando la etiqueta de versión en el campo selector.

En la primera consola vamos a realizar los despliegues de nuestras aplicaciones, para ello ejecutaremos el siguientes comando.

    kubectl apply -f app-v1.yaml

En la segunda consola observaremos como el cluster de k8s realiza el despliegue de las aplicaciones:

    watch -n 1 kubectl get pod
    
En la tercera consola observaremos la información emitada por la aplicación a través del endpoint expuesta por ella.

    while sleep 0.1; do curl {ip}; done

**Nota:** La dirección ip corresponde al servicio creado en el despliegue.

Por ultimo, en la primera consola desplegamos la versión 2.0.0 de la aplicación y en las otras dos consolas se evidenciará como se construye la nueva versión de la aplicación y como esta interactua con la versión 1.0.0

    kubectl apply -f app-v2.yaml

Para redireccionar el tráfico entre la versión 1.0.0 y la versión 2.0.0 se hará uso de los siguientes comandos:

    kubectl patch service fruit-service-bg -p '{"spec":{"selector":{"version":"v2.0.0"}}}'
    kubectl patch service fruit-service-bg -p '{"spec":{"selector":{"version":"v1.0.0"}}}'
    
Después de evidenciar esta estrategia de despliegue, se recomienda eliminar todos los componentes creados en el cluster, para ello ejecutar los siguientes comandos.

    kubectl delete app-v1.yaml
    kubectl delete app-v2.yaml
