
# Documentación prueba  

Con este documento se espera poder dar a entender cada una de las consideraciones tenidas al momento de diseñar y desarrollar la aplicación solicitada. 
  
## Arquitectura y Diseño de la nube
Se selecciona [AWS](https://aws.amazon.com/es/) como cloud proveedor. 
Para esta etapa se diseña una solución en la nube la cual permite tener un sistema altemente disponible y escalable. Se espera disponibilizar un Route53 el cual es un DNS que nos permitirá tener un dominió el cual será la puerta de entrada a 
la aplicación, seguido de un NAT Gateway, que es el encargado de permitirno conexiones hacia la subred publica. 

En Subred publica contendrá el CDN (CLoudfront) que permite disponibilizar el contenido estatico (App frontend) alojado en un Bucket de S3, adicionalmente cuenta con un WAF(Web Applicaiton Firewall) que nos permite evitar ataques de Inyeccion de SQL, XSS (Cross site scripting), bots y otros. Seguidamente estará el API Gateway el cual validará la autenticación del usuario por medio de cognito (se espera disponibilizar una serie de lambdas que permitan interactuar con los servicios de congnito) y el cual se encargará de redirigir los recursos a los servicios correspondientes.

En la subred privada se disponibilizará por un lado un balanceador de carga el cual balanceará las peticiones entrantes entre las diferentes instancias EC2 existentes en el Auto Scaling Group, el cual se espera logre crear instancias dependiendo de la configuración realizada. Por otro lado, se tiene una lambda, la cual al ser serverless se espera que se disponibilice a medida de que sea requerido, permitiendo tener disponiblidad y escalabilidad sobre la misma.

Cada una de estos servicios consultarán en una base de datos en memoria Redis, la cual permitirá tener información de una manera más rapida, esta BD será actualizada por medio de streams de Dynamo (No está plasmado en diseño) generando eventos al momento de realizar inserciones sobre las tablas Orders y Products, este stream accionará una lamda que se encargará de actualizar la BD en memoria.
  
## DevOps

Se implementan pipelines de CI/CD en GitHub Actions en el repositorio de insfraestructura, el cual es desarrollado con Terraform y el cual es configurado para que al momento de realizar un PR hacia la rama main este ejecute el Terraform init y el plan y una vez sea aprobado y fusionado el cambio del PR este ejecute el apply de la infraestructura (Por cuestiones de tiempo se entrega el Pipeline no funcional). Este mismo proceso se espera hacer para los repositorios donde se alojan los servicios que serán desplegados en instancias EC2 y las Lambdas.
Repositorio de infraestructura: [LinkTic Infraestructure](https://github.com/andres043/linktic-infra)
 
## Backend

### Orders API
El proyecto [orders-api](https://github.com/andres043/linktic-orders-api) contiene el endpoint para listar las ordenes. Este servicio fue creado en Java, usando el framework spring y siguiendo la arquitectura hexagonal.

### Products API
El proyecto [products-api](https://github.com/andres043/linktic-products-api) contiene el endpoint para listar y crear productos. Este servicio fue creado en Java, usando el framework spring y siguiendo la arquitectura hexagonal.

Estos dos servicios serán desplegados en instancias EC2, posteriormente se espera que utilice el servicio de Cloudwatch y que el endpoint de Listar productos se conecte a Redis para poder suministrar la informacion de los productos existentes de una manera mas rapida y eficiente. 

### Create Order
El proyecto [create-orders](https://github.com/andres043/linktic-create-orders) contiene el endpoint para crear ordenes. Este servicio fue creado en Golang, usando Wire y siguiendo la arquitectura hexagona.

Este endpoint espera disponiblizarse por medio de una lambda, debido a su rapido procesamiento y despliegue se considera una buena alternativa para usar lambda.

Al final se espera que estos servicios mencionados anteriormente utilicen los servicios de Cloudwatch para el manejo de logs.

De igual manera se espera tener las lambdas faltantes las cuales cubririan los flujos de autenticación, autorización y eventos de Dynamo Stream.

## Frontend
Para este desarrollo se creó el proyecto [Frontend](https://github.com/andres043/linktic-frontend), para el cual incialmente se pensó desarrollar en el framework React de Javascript, pero por cuestiones de tiempo y curva de aprendizaje, se optó por desarrollarlo en Javascript y HTML usando Bootstrap.
