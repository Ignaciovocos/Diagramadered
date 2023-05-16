# Consigna:
Produzca un diagrama de red (puede utilizar
lucidchart) de una aplicación web en GCP o AWS y escriba una descripción de
texto de 1/2 a 1 página de sus elecciones y arquitectura.
El diseño debe soportar:
• Cargas variables
• Contar con HA (alta disponibilidad)
• Frontend en Js
• Backend con una base de datos relacional y una no relacional
• La aplicación backend consume 2 microservicios externos
El diagrama debe hacer un mejor uso de las soluciones distribuidas.

# Punto 1 : Diagrama de red para aplicación en AWS



# Stateful web application Ignacio Vocos


Bestsneakers.com es una aplicación web que permite a los usuarios poder comprar zapatillas de forma online,tiene un carro de compras, muchos usuarios se conectan y navegan el sitio web al mismo tiempo.Necesitamos poder escalar verticalmente, mantener la escalabilidad horizontal y mantenerlo lo más lo stateless posible. Fundamental que los usuarios no pierdan lo que han seleccionado en el carro de compras mientras navegan el sitio web y que puedan tener sus propios usuarios ( domicilio, preferencias, id profile, foto perfil, etc) en una base de datos.


|                                            Diagrama de Red en Amazon Web Services                                        |


Route 53 


-Seteamos un R53 donde el website URL es api.bestsneakers.com con un “alias record” para que lleve al usuario directamente al LB. Gracias a esto no necesitamos una elastic IP atada a cada instancia ademas solo hay 5 elastic ip por region y por cuenta default.


Usuarios

-Se conectan a través de R53 y son automáticamente dirigidos a un LB con multi AZ para que la carga pueda ser bien distribuida a las distintas instancias. Garantizando una buena performance y experiencia en el sitio web. Le agregamos Health checks para que no envie informacion a una instancia que se encuentra caida por ejemplo.


EC2 Instance

-M5 en diferentes availability zones dentro de un auto scaling group el cual puede escalar horizontal como verticalmente de acuerdo a la demanda. Podemos utilizar “reserved instances” para garantizar un mínimo de instancias funcionando por ej 1 EC2 en cada availability zone y así poder economizar gastos a través de ese plan de pago.


Data

-Necesitamos que cada usuario pueda conservar sus datos cada vez que inicia sesión, para ello utilizaremos un “Server session” con un “Sesion ID” para cada usuario. En el background utilizaremos un Elasticache Cluster entonces la EC2 por ej, va a agregar el contenido del carro de compras al elasticache como así también el session id para que cuando nuestro usuario haga un segundo ingreso con su session id y vaya a otra instancia EC2 se vincule la data vinculada al usuario. Elasticache tiene un sub milisegundo performance lo cual hace que sea realmente rápido y seguro ya que los atacantes no pueden cambiar lo que está allí alojado.

-Necesitamos también almacenar los datos del usuario en una base de datos relacional, en este caso mi EC2 va a dirigirse a un RDS el cual es bueno ya que es especial para almacenar por mucho tiempo y requerir la data como ser el nombre, la dirección, etc.Para darle menos tráfico a RDS bajando el uso de CPU y aumentando así la performance podemos utilizar “lazy loading” porque la gran mayoría de los usuarios navega el sitio web leyendo información ( precios, descuentos, novedades, opiniones, reputación , etc) lo que hace esto es  que nuestro usuario se dirige a la EC2 mira en el elasticache y dice , ¿tu tienes esta info? si no la tiene lo que va a hacer es leerla de la RDS y ponerla en el caché nuevamente para que la información quede cacheada para ello debemos implementar un buen mantenimiento del caché.

-Podemos agregar también un DynamoDB ya que es un servicio de base de datos NoSQL totalmente administrado que puede escalar automáticamente para manejar cargas de trabajo que cambian rápidamente y grandes volúmenes de datos. (procesamiento de imágenes, videos, transacciones financieras en tiempo real, servicio de mensajería y chat entre los usuarios etc).


Frontend

El Frontend está construido con JavaScript y se sirve directamente desde las instancias EC2. No hay un servidor web dedicado para servir el frontend.


Api Gateway / backend

El API Gateway actúa como una puerta de enlace para todas las solicitudes de API entrantes, que se enrutan a los microservicios relevantes en el backend de la aplicación. La API Gateway también realiza funciones como autenticación, autorización, enrutamiento, transformación y caché de solicitudes.
En el backend de la aplicación, hay dos microservicios externos que en este caso puede ser uno para manejar pagos (API Paypal) y otro para manejar el envío de los productos (API DHL) que se consumen para realizar tareas específicas. Estos microservicios se comunican con la aplicación a través del API Gateway.


Funcionamiento constante

-Para poder garantizar un correcto y continuo funcionamiento ya de por sí Route 53 es highly available, nuestro ELB es Multi AZ y nuestras distintas instancias se encuentran alojadas en distintas zonas, para poder lograr que ante por ejemplo un eventual desastre natural nuestra app web pueda mantener su correcto funcionamiento.


Seguridad

-En cuanto a seguridad queremos estar seguros por ello podemos abrir el tráfico HTTP/HTTPS desde cualquier lugar desde el lado de ALB. Después podemos restringir el tráfico que llega de las EC2 desde ELB y restringir tráfico en Elasticache y RDS proveniente de los EC2 security groups.
                                                        

# 5 Pilares de la correcta aplicación de una arquitectura


Cost: Escalando nuestras instancias de forma vertical de un T2 micro a un M5, Usando un Auto Scaling Group para tener la cantidad correcta de instancias de acuerdo a la carga, como así también reservar el mínimo de instancias necesarias para optimizar costos.


Performance: Poder escalar verticalmente, ELB elastic load balancer, autoscaling groups básicamente cómo podemos mejorar la performance de nuestra app con el tiempo.


Reliability: Como Route 53 puede ser usado para redirigir el tráfico para a las distintas EC2 instancias, usando Multi AZ para ELB y como así también para ASG.


Security: Como hemos configurado los grupos de seguridad para linkear el load balancer a nuestras instancias de forma confiable.


Operational excellence: Cómo podemos pasar de un proceso manual a tener todo automatizado con autoscaling groups etc.

