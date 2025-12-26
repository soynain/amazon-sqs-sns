# amazon-sqs-sns

En este repositorio abordaremos implementaciones básicas de SQS y SNS.

Cabe aclarar con respecto al repo de la práctica de kafka que está en la elección escoger
donde hostear tu servicio de KAFKA, ya sea en un EC2, en un MSK de amazon o en un pod de kubernetes.

Cuestión de costos y arquitectura.

Implementaremos el SDK de SQS y SNS tomando como base los dos micros de la práctica de kube y docker.


# Avances 25/12/2025 

Después de sufrir una indigestión por comer costillas, decidí para no empacharme darme un buseo nocturno por la documentación de AWS.
El único desafio fue confirmar si estaba configurando bien mis credenciales para el CLI, además de batallar con el POM (sorry por chatgepetear en eso).

De ahí en fuera la conexión hasta ahora en micro A es super sencilla. Creo es menos batalloso este tema que Kafka honestamente, traeré updates más adelante.

Configura un IAM user con access key, con el permiso AmazonSQSFullAccess y la politica anhidada PowerUserAccess

<img width="1061" height="1041" alt="image" src="https://github.com/user-attachments/assets/57b002b9-ffa9-44cd-be82-171a1549bcc8" />

En la carpeta de windows en este caso, solo asegurate que se te generaron bien las credes

<img width="1024" height="296" alt="image" src="https://github.com/user-attachments/assets/4afe0eb8-fd70-40de-8295-bf9bbf610965" />

También puedes comprobar tus access key con el comando aws sts get-caller-identity.

La gracia cae sobre la configuración, en una instancia ECS las credes se guardan en otro directorio de Linux y para el SDK es importante configurar más que nada
algo denominado credenciales temporales. A nivel Kubernetes eso se hace con IAM IRSA, que se configura con un tag en el yaml del type: ServiceAccout. Eso o rotarlas manual, pedirselo eso
al encargado de infra si tienen una organización bastante defragmentada.

De ahí en fuera, parece ser más sencillo que el mismo Kafka, y claro no tiene la misma potencia:

<img width="1865" height="829" alt="image" src="https://github.com/user-attachments/assets/5c4c94e7-6e29-4357-8c18-f99f67f1d8e4" />

Diferencias de Kafka y SQS

*Kafka aguanta más datos y concurrencia, diseñado para alto trafico, y guarda los mensajes por intervalos largos de tiempo, ideal en etls también.
*Retry's nativos para envio de mensaje, poca latencia si no activas los ACK's.

SQS
*Ideal para lambdas o procesos chiquitos/medianos, colas sencillas que se comunican con el ecosistema de AWS directamente, haciendolo
una implementación del propio stack de AWS, puedes usar S3, y cosas asi.
*No retiene por mucho tiempo los mensajes.
*Pueden perderse los mensajes, existe ese riesgo.

<img width="580" height="183" alt="image" src="https://github.com/user-attachments/assets/c13b0545-fabc-4f91-9e15-1d721437e711" />

Haré solo un envio y recepción.

# Avances 25/12/2025 parte 2
Modifiqué mi código inicial e hice unos endpoints sencillitos, uno para crear una cola, el que hice para consultar todas las colas:

<img width="1048" height="169" alt="image" src="https://github.com/user-attachments/assets/a32e21a6-4610-4106-8d04-7e37df7f98d5" />

Y otro para mandar un mensaje, aqui he estado leyendo un tópico interesante que es el tema del long polling, estuvo enredoso más que nada por la implemehnjtación
de un worker sencillo, ya que no funciona por tcp, si no por pollings esta herramienta:


<img width="1587" height="563" alt="image" src="https://github.com/user-attachments/assets/ebb2d883-5ec0-49ff-90da-ea99ce18aa14" />

<img width="1546" height="708" alt="image" src="https://github.com/user-attachments/assets/707d53d0-eb6c-4e3b-a92d-5f1879f05a1f" />

Es importante repasar el polling nativo. Creo el .waitTimeSeconds(20) detiene el hilo del mismo worker, de esos se encarga la libreria por debajo.

Desde microa mandas el mensaje

<img width="1694" height="178" alt="image" src="https://github.com/user-attachments/assets/63bf1c65-8083-43db-b217-f70ecb84ce5d" />

<img width="1603" height="910" alt="image" src="https://github.com/user-attachments/assets/dd60af84-e15c-46c3-bef4-b44f2ff37544" />

Y listo. Si tiene más complejidades de acuerdo al caso de uso pero no necesitamos extender el tópico. En si, ya hay un repo
con fragmentos del codigo de acuerdo a tu caso de uso, aquí: https://github.com/awsdocs/aws-doc-sdk-examples/blob/main/javav2/example_code/sqs/src/main/java/com/example/sqs/SQSExample.java#L151

Y en la docu de AWS SDK para SQS, ya hay escenarios listados: <img width="296" height="671" alt="image" src="https://github.com/user-attachments/assets/bd1b48bf-2821-4d8e-8d43-e15437a20404" />

Con esto damos por concluido el apartado de SQS, pero si hay que leer y a lo mejor al adentrarnos al SNS, validar el tema del worker.

Otra cosa, ahí harcodeamos el worker con el url pero también podemos pasarles parametros, por medio de un hashmap, me abstentaré a hacerlo, lo investigué
chatgepeteando. Los temas de concurrencia creo que serán muy importantes en el futuro.... Asyncs y threads. No hay pex.

Pasaremos con SNS. Ah otra cosa, hay una dependencia que es la spring cloud aws starter, esa no es compatible con Spring boot 4.0.o aun, si usas
spring boot 3.5.+ esa libreria te hará las cosas mucho más fácil.

# Avances 26/12/2025

Ahora le seguimos con SNS, no haré una susbscription A2A, la haré a2p, porque es más orientado al fanout queue.
El chiste en estas prácticas es entender los fundamentos de la síntaxis. No crear ecosistemas completos exceptuando la práctica
de kubernetes. Solo la extenderemos si me rechazan en una oferta que ando persiguiendo.

Creamos unos endpoints para creación del tópico

<img width="1754" height="1330" alt="image" src="https://github.com/user-attachments/assets/b16eea9b-75c8-4f29-aeef-34466891a19a" />

<img width="1410" height="933" alt="image" src="https://github.com/user-attachments/assets/6dbe42cf-0665-4d66-ba32-7feb554fa6da" />


Responden bien

<img width="935" height="214" alt="image" src="https://github.com/user-attachments/assets/c9641173-3438-4460-9253-40b71d5ba9a9" />

<img width="1283" height="151" alt="image" src="https://github.com/user-attachments/assets/8cba6781-7434-4805-948b-e82165109f7e" />

Tristemente y al parecer el plan gratuito no te deja mandar mensajes sms. 

Con esto acabamos la práctica de la sintaxis sencilla.......... bueno, veré si hago el fanout queue para profundizar y de ahí a leetcodear un poco más.

Ahora dominamos la nube. Faltaria también Flink.
