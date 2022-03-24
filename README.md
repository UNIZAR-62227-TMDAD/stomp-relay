# STOMP broker relay

Se ha configurado este ejemplo para que Spring Boot haga un relay a un message broker que soporte STOMP.

Se ha elegido RabbitMQ porque soporta STOMP. 
Vamos a utilizar Docker para tenerlo en local. 
Sin embargo,
como no podemos estar seguros de que el [plugin de STOMP](https://www.rabbitmq.com/stomp.html) esté activado,
vamos a crear una imagen donde estemos seguros de su activación:

```Dockerfile
FROM rabbitmq:3-management
RUN rabbitmq-plugins enable --offline rabbitmq_stomp
EXPOSE 61613
```

Y nos aseguramos que en el correspondiente `docker-compose.yml` estén abiertos los puertos:

```yml
version: "3.9"
services:
  amqp:
    build: .
    ports:
      - "61613:61613"
      - "15672:15672"
      - "5672:5672"
```

El adaptador de STOMP usa el puerto 61613 y utiliza como usuario y contraseña `guest`. 
Por tanto, la configuración de `WebSocketConfig` debe ser actualizada a:

```java
config.enableStompBrokerRelay("/topic")
        .setRelayHost("localhost")
        .setRelayPort(61613)
        .setClientLogin("guest")
        .setClientPasscode("guest");
```

Para ejecutar este código, lo primero que hay que hacer es arrancar RabbitMQ. 
En http://localhost:15672/ podemos acceder con `guest`/`guest` al administrador. 

Si ejecutamos el test de integración (`GreetingIntegrationTests`) que ha sido modificado para ejecutarse 100 veces,
podremos ver que efectivamente se está utilizando RabbitMQ,
via el exchange `amq.topic` con una cola cuyo nombre comienza por `stomp-subscription`. 

Finalmente, 
si arrancamos la aplicación y vamos a http://localhost:8080/, 
podremos comprobar que los mensajes que enviamos pasan también por RabbitMQ. 
Cada vez que un cliente hace una suscripción, el relay crea una suscripción en RabbitMQ.
