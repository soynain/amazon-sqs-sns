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


# Conclusiones y comparativa

Cuando aprendemos Kafka tendemos a poner atención en la herramienta pero no en su propósito real, si es una cola de eventos
con particiones que respetan el orden de acuerdo al tópico, sin embargo esto funciona en sistemas
event-driven, los eventos al ser enviados se quedan como logs sobre el handler de kafka. 

¿Qué quiere decir esto? que, en un contexto event driven, el log se vuelve el eje sobre el sistema. Se vuelve un punto donde puedes tracear
ese log, al igual que sus anteriores predecesores del log, a esto se le llama contratos y versionados de contratos. 

Esos logs con contratos, deben ser inmutables, y versionados para reprocesados, de esa manera te permite detectar si hubieron errores previo a algún procesado superior o inferior.

Es un log auxiliar, complementario a lo mejor a los logs tradicionales de micros que pudieras vincular con Kibana. Tu evento se vuelve el eje central.
Claro, piensa en las particiones, y en como auditarlo más adelante, tal vez respaldando batches de logs en almacenamientos S3 de manera anual o por mes y/o periodo, o al corte del año
borrarlos y tener espacio para nuevos, pero depende de las necesidades de auditoria y que tanto necesites esos logs. Son temas para sistemas REALMENTE GRANDES Y AUDITADOS.

Los esquemas los puedes hacer por json normal, o por Avro y Protobuf, son dos herramientas externas que te ayudan
a hacer esa configuración más gráfica. Solo lo profundizaré cuando llegue lejos en la oferta laboral que busco.

A ciencia cierta, la diferencia radica entre los tipos:
BACKWARD COMPATIBILITY:
Producer v2  →  Consumer v1   ✅
Producer v1  →  Consumer v2   ✅

✔️ Permitido
Agregar campos opcionales

Agregar campos con default

Quitar campos solo si no eran obligatorios

❌ No permitido
Agregar campos obligatorios

Cambiar tipos

Renombrar campos

FORWARD COMPATIBILITY
Producer v1  →  Consumer v2   ✅
Producer v2  →  Consumer v1   ❌

✔️ Permitido

Quitar campos

Ignorar campos antiguos

Mantener tipos compatibles

❌ No permitido

Agregar campos que el consumer espera siempre

Cambiar semántica


Ahora con SQS/SNS los casos de uso cambian:
Son colas más sencillas, directamente más conectadas con el ecosistema de AWS directamente, ofreciendote
una cola sencilla para comunicación de tus servicios, ¿diferencia? los eventos no se guardan. Solo se entregan una vez y expiran.

Por ejemplo analicemos el código de los mismos ejemplos de AWS, pero antes aclarar: SQS/SNS no guarda nada. SQS tiene un visibility time, al obtener un mensaje
ese tiempo transcurre y tendrás ese tiempo para volver a traer ese mensaje. Cuando lo borres, dentro del margen del visibility desaparece de la cola. 
Pero, a diferencia de kafka, no tiene retrys, es solo una vez!! amazon te puede dar los ids de los eventos que se perdieron y tu armas tu retry si algunos
envios fallaron, algo que kafka facilita en cierto sentido.

Aquí por ejemplo en el repo oficial lo muestra más que claro, tu programa tus retrys, tu impondrás ese control:

<img width="1005" height="896" alt="image" src="https://github.com/user-attachments/assets/e86efa0a-36bf-468d-9a02-d395c0fc7b36" />

Y como se constato antes, por workers se obtienen los mensajes, con long pulling, un hilo que se mantiene despierto por x tiempo para ahorrar costos. Eso va en coordinación con el delay de tus mensajes que configures
y el tiempo de espera. También SQS te permite implementar el patrón fifo para que respete el orden. Kafka respeta el orden POR PARTICIONES. FIFO no admite duplicados tampoco.

SQS también solo sigue una regla: un mensaje solo corresponde a un consumidor, no es FANOUT, si tienes multiples producers, no todos recibirán el mismo mensaje por ese hecho.

Ahora, SNS! es un servicio de PUBSUB, más que de cola. Manda mensajes a consumers que pueden ser A2A o A2P. Puedes mandar eventos para desencadenar SMS, correos, lambdas,
sms, y SQS's también, por eso se dice que puedes replicar el fanout si fusionas SNS /SQS. En sistemas medianos queda de perlas.

SNS solo procesa si mandó el mensaje o no, a diferencia de SQS que si espera una confirmación de eliminación, muy importante tomar esta diferencia  y KAFKA depende de ti cuando tiempo los retengas.

Creo era lo más coherente hacer esta reflexión final no solo por el tema de la sintaxis sencilla y la práctica fácil, pero para saber en que casos hay que usar estas herramientas.

Las documentaciones de AWS ya tienen esos escenarios. SNS usalo en notificaciones 
<img width="1463" height="871" alt="image" src="https://github.com/user-attachments/assets/e9fce89b-c996-4703-8257-2b2b95c2c48f" />

SQS usalo en, batches de envío y eventos sencillos/medianos entre servicios o micros:

<img width="1310" height="1023" alt="image" src="https://github.com/user-attachments/assets/71aefa37-1a45-486c-b501-6aa7f3d790b2" />

Aquí se muestra un ejemplo sencillo para ello

<img width="1001" height="657" alt="image" src="https://github.com/user-attachments/assets/fa154c7a-6ccb-4ac9-aa95-74cd03cffec1" />

En SQS puedes mandar BINARIOS y objetos de S3!!!

<img width="998" height="425" alt="image" src="https://github.com/user-attachments/assets/3222d350-ffd7-4fcf-8e17-04dcff754080" />

Con SNS solo orquestas notificaciones de arranque.

Piensa en esas posibilidades, no mandarás binarios por kafka, pero si por estos servicios. 
El patrón inicial es SQS/SNS o RABBITMQ, redis o hazelcast a lo mejor.

# Con esto se concluye las reflexiones

No profundizaremos más, hasta después... de alcanzar ese valhalla.






