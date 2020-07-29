---
published: true
title: "Instancia de MongoDB en la nube gratis"
cover_image: 
description: 
tags: mongodb
series: mongodb
canonical_url:
---

La gente de MongoDB facilita el uso de todo un clúster de forma gratuita dándonos de alta en su página https://www.mongodb.com/cloud/atlas

![image-20200626163915088](assets/mongo-1.png)

Tenemos que darnos de alta en la plataforma, pero fíjate que no te van a pedir la tarjeta de crédito. La forma más rápida es pulsar para logarnos con nuestra cuenta de Google.

![image-20200626164239353](assets/mongo-2.png)

Independiente del proceso de registro, al final tenemos que caer en esta pantalla donde podemos empezar a crear el clúster.

![image-20200626164544189](assets/mongo-3.png)

## Dando de alta el clúster

Realizada está selección se nos muestra un asistente donde en primer lugar vamos a poder seleccionar el proveedor de cloud y la región, aquí el consejo es coger una región lo más próxima a dónde se vaya consumir la instancia por aquello de los tiempos de latencia.

![image-20200626170048842](assets/mongo-4.png)

En el siguiente paso "Cluster Tier" dejamos el que viene seleccionado por defecto que como indica será "Free forever".

![image-20200626170226065](assets/mongo-5.png)

Como ves en las características, para ser un recurso gratuito, son extremadamente generosas.

En los dos últimos pasos, nos informa que con esta versión de clúster no tenemos posibilidades de backup y, en el último paso, podemos establecer un nombre a nuestro clúster.

![image-20200626170440316](assets/mongo-6.png)

Una vez seleccionados todos los pasos, podemos darle al botón "Create Cluster" que se mantiene en el footer de la página.

![image-20200626170607574](assets/mongo-7.png)

Hecho esto el sistema nos informa del checklist que nos falta por completar. 

![image-20200626172415153](assets/mongo-8.png)

## Creando los primeros usuarios

Pinchando en la opción "Create your first database user" el sistema nos muestra donde tenemos que hacerlo, en "Database Access".

![image-20200626172724699](assets/mongo-9.png)

Pinchando sobre la opción nos aparece la siguiente pantalla, donde pinchamos en el botón "Add New Database User"



![image-20200626172850591](assets/mongo-10.png)

Al pinchar nos mostrará la siguiente pantalla donde podemos establecer las credenciales para el acceso y los privilegios de "Atlas Admin".

> Podemos repetir estos pasos para añadir más usarios con otros privilegios.

Pudiendo quedar de esta forma, aunque dependerá de las necesidades que tenga cada persona. En mi caso un usuario admin como administrador de todo el clúster; y un usuario master para poder operar con las bases de datos y las colecciones:

![image-20200626173555680](assets/mongo-11.png)

## Preparando la conexión remota

Para poder conectar con la instancia desde una aplicación o cliente de MongoDB remoto, necesitamos ir a la sección "Clusters" y pulsar en el botón "CONNECT".

![image-20200627225851338](assets/mongo-12.png)

Al pulsar sobre este botón, nos muestra una pantalla donde dice que nos falta configurar más cosas para permitir el acceso remoto y nos ofrece la posibilidad de añadir una lista de IPs permitidas.

![image-20200627230122714](assets/mongo-13.png)

Pulsamos en el botón "Add your Current IP Address", con lo que se despliega una ventana, donde aparece nuestra IP actual, nos permite añadir una descripción y un botón "Add IP Address"

![image-20200627230354960](assets/mongo-14.png)

Pulsando en este botón, se nos informa de que ya estamos listos para conectar y podemos pasar al siguiente paso "Choose a connection method".

![image-20200627230554321](assets/mongo-15.png)

En el siguiente paso nos ofrece tres posibles tipos de conexión:

* Connect with the mongo shell: si queremos hacerlo utilizando la interfaz interactiva de JavaScript que te ofrece mongo.
* Connect your application: si lo que queremos es conectar una aplicación nuestra.
* Connect using MongoDB Compass: si queremos utilizar el cliente Compass para conectar.

![image-20200627231017712](assets/mongo-16.png)

En mi caso voy a seleccionar la opción del cliente de Compass, en la siguiente pantalla nos permite descargar este cliente, si no lo tenemos ya y, sobre todo, poder copiar la URL de conexión que necesitamos en el siguiente apartado.

![image-20200627231333105](assets/mongo-17.png)



## Conectar desde Compass

Para comprobar la conexión y poder operar con esta instancia vamos a utilizar la herramienta "MongoDB Compass" la cual podemos descargar e instalar en cualquier sistema operativo desde esta URL (https://www.mongodb.com/try/download/compass) si no lo hemos hecho desde el asistente del anterior apartado.

Al abrir Compass se nos muestra una ventana donde poder crear una nueva conexión, y donde tendremos que copiar la URL copiada del anterior apartado, aportando un usuario y password válidos.

![image-20200627231737145](assets/mongo-18.png)

Si todo es correcto, al pulsar en "CONNECT" entraremos en la administración del cluster pudiendo crear nuevas base de datos y colecciones.