# Proyecto final: chat multithreading 

Hola, bienvenidos al proyecto final, este proyecto necesitará que logren varios objetivos, posteriormente deberán buscar la manera de que dichos proyectos/lógica sean unidos para lograr el cometido, aquí solo se irán dando algunos `hints`, los cuales podrán ser usados para lograr el cometido. Cabe destacar que el orden no es importante, ya que lo "_difícil_" realmente es unir todo de una forma amena y **lógica**, sin más dilatación, a lo que venimos.

# 1 - Objetivo I: lograr una conexión a la base de datos y recuperar información usando java

## 1.1 - Instalar MySQL
Para empezar, deberán ver el vídeo donde hago la conexión a la base de datos, no necesariamente deben seguir lo que hay en el vídeo, aconsejo más indagar en las distintas formas que hay para hacer una conexión a la base de datos, recomiendo que trabajen sobre un sistema linux, así no tendrán que hacer las configuraciones adicionales que yo di en el vídeo, a continuación dejo algunos enlaces donde hay tutoriales que les pueden ayudar a instalar MySQL en sus computadoras.


1. [Instalar MySQL de forma nativa en windows](https://dev.mysql.com/doc/refman/8.0/en/mysql-installer.html)
2. [Instalar MySQL en windows (vídeo)](https://www.youtube.com/watch?v=AP-LloZZhTM)
3. [Instalar MySQL en WSL ](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-database)
4. [Instalar MySQL usando docker](https://www.datacamp.com/tutorial/set-up-and-configure-mysql-in-docker)


## 1.2 - Dar de alta la base de datos y las tablas
Una vez que hayan instalado MySQL, deberán dar de alta la base de datos con la configuración adecuada, recuerden que deben seguir las instrucciones del vídeo y ejecutar un script para levantar la base de datos "`chat_db`", así como también la tabla de `usuarios` y `mensajes`.

El script necesario para dar de alta la tabla y los registros es [el siguiente](https://gist.github.com/software-alchemist-official/494af29d7f751ae59717238b42bb7b67).

## 1.3 - Entender las sentencias SQL

Entender las principales sentencias SQL, realmente no es necesario que a este nivel dominen SQL, pero no está de más que sepan cuáles son las principales sentencias, en especial las operaciones CRUD, dicho de otra forma: creación, lectura, actualización y eliminación de registros en la base de datos. Pueden leer el [siguiente artículo](https://brandominus.com/blog/creatividad/manual-sentencias-basicas-en-mysql/) para darse una empapada.

## 1.4 - Crear el proyecto Java y los servicios

Siguiendo la lógica del vídeo, ahora deberán crear su propio proyecto de java, repetir las acciones tal y como las hice yo o bien consultar en fuentes externas, deberán crear al menos dos servicios UserService (encargado de tener los métodos para las operaciones CRUD en la tabla de usuarios) y MessageService, lo mismo pero para la tabla de mensajes.

Yo aconsejo que el servicio de usuarios **por lo menos** los siguientes métodos:

1. Guardar un usuario en la base de datos
2. Recuperar un usuario usando el ID.
3. Recuperar todos los usuarios.
4. Recuperar el usuario por username
5. Recuperar el usuario por email

Son libres de agregar otras dinámicas, eso es la milla para ustedes, pero para lo que vamos a hacer esos tres deberían ser suficientes.

Para el servicio de mensajes (MessageService) aconsejo estos métodos:

1. Guardar un mensaje
2. Recuperar un mensaje por ID
3. Recuperar los últimos 10 mensajes de la base de datos (este ayudará a una posible funcionalidad de que cuando un usuario entre al chat vea los últimos mensajes y entienda el contexto de la conversación, aunque esa funcionalidad es un extra y no es parte del proyecto).

# 2 - Objetivo II: entender cómo funcionan los clientes.

Este es fácil, ver el primer vídeo de sockets, en ese vídeo se muestra como crear un cliente, por favor jugar con los métodos disponibles, conectarse al servicio de Star Wars, lograr ver la película, etc. El objetivo es que el lado del cliente sea fácil para ustedes.

Hagan esto en un proyecto Java independiente, intenten no abrumarse poniendo todo junto, ya que podrían llegar a tener conflictos.

# 3 - Objetivo III: crear un proyecto server

Este objetivo no es multi hilo, sencillamente generen un proyecto servidor de un solo hilo e intenten ejecutar el servidor y posteriormente varios clientes (recuerden repasar el vídeo para que sepan como hacer para tener instancias paralelas corriendo en el mismo IDE), como un plus: generen los archivos `.jar` y ejecuten el jar del server y posteriormente algunas instancias del .jar del cliente, todo en terminales separadas.

# 4 - Objetivo IV: crear un cliente-servidor multi hilo

Con el conocimiento adquirido hasta estos momentos, ya deberían ser capaces de generar otro proyecto Java, pero que maneje múltiples clientes, recuerde repasar el vídeo y usar algún `ExecutorService`, ir a la API, usar Google y ChatGPT para ver qué executor services tienen a su disposición, o bien, pueden usar el que uso en el vídeo, no pasa nada. Este sería ya el paso final para comprobar que tienen todo el conocimiento para abordar la parte final.

# 5 - Objetivo V: Indagar de cómo lograr el chat sin usar una base de datos.

Antes de integrar la base de datos, sería excelente que intenten lograr el chat sin meter la base de datos, **dejo estos dos pequeños snippets cómo referencia (investiguen todo lo que sepan y pidan a chatGPT o google más ejemplos)**:


**ChatServer.java**
```java 
package com.software.alchemist.model;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.logging.Level;
import java.util.logging.Logger;

public class ChatServer {

    private static final int PORT = 12345;

    private static final Set<ClientHandler> clientHandlers = Collections.synchronizedSet(new HashSet<>());

    private static final Logger LOG = Logger.getLogger(ChatServer.class.getName());

    public void init() {

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {

            LOG.log(Level.INFO, "Se ha iniciado el server en el puerto: {0}", PORT);

            while (true) {
                Socket socket = serverSocket.accept();
                ClientHandler clientHandler = new ClientHandler(socket);
                clientHandlers.add(clientHandler);
                LOG.log(Level.INFO, "Se ha agregado el usuario {0} a los clientes", clientHandler);
                new Thread(clientHandler).start();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void broadcastMessage(String message, ClientHandler sender) {
        synchronized (clientHandlers) {
            clientHandlers.stream()
                    .filter(clientHandler -> clientHandler != sender)
                    .forEach(clientHandler -> clientHandler.sendMessage(message));
        }
    }

    public static void removeClient(ClientHandler clientHandler) {
        synchronized (clientHandlers) {
            clientHandlers.remove(clientHandler);
            broadcastMessage(clientHandler.getUsername() + " ha salido del chat.", clientHandler);
        }
    }


}

```


**ClientHandler.java**
```java
package com.software.alchemist.model;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.util.logging.Level;
import java.util.logging.Logger;

import static java.util.Objects.nonNull;

public class ClientHandler implements Runnable {

    private final Socket socket;

    private PrintWriter out;

    private String username;

    private static final Logger LOG = Logger.getLogger(ClientHandler.class.getName());


    public ClientHandler(Socket socket) {
        this.socket = socket;
    }


    @Override
    public void run() {

        try {

            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            out = new PrintWriter(socket.getOutputStream(), true);
            username = in.readLine();
            ChatServer.broadcastMessage(username + " se ha unido al chat.", this);

            LOG.log(Level.INFO, "{0} se ha unido al chat.", this);

            String message;
            while (nonNull((message = in.readLine()))) {
                System.out.println(username + ": " + message);
                ChatServer.broadcastMessage(username + ": " + message, this);
            }

        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            ChatServer.removeClient(this);
        }

    }


    public void sendMessage(String message) {
        out.println(message);
    }

    public String getUsername() {
        return username;
    }

    @Override
    public String toString() {
        return "ClientHandler{" +
                "username='" + username + '\'' +
                '}';
    }
}

```

Por favor logren un chat con pura consola y sin MySQL, pues eso es prácticamente en proyecto, el guardar los mensajes es lo de menos, así como lograr registrar usuarios, etc.

# **HASTA ESTE PUNTO ES MÁS QUE SUFICIENTE COMO PROYECTO FINAL PUES YA TENDRÍAN UN CHAT FUNCIONAL, TODO LO DEMÁS (LA INTEGRACIÓN CON LA BASE DE DATOS ES UN PLUS, ASÍ COMO LOS MENÚS E INTERACCIONES)**




# 6 - Instrucciones finales.

Para poder lograr entender el flujo, intenten ponerse en el papel del cliente, ustedes van a ejecutar algún proyecto llamado chat-cliente, el cual deberá basarse en lo que aprendieron en la creación de clientes, el hostname al que se conectará es a su propio localhost, pregunta: ¿qué es lo primero que debería ver el cliente al conectarse?

Aquí quiero que jueguen, podrían saludarlo, esperar una tecla para continuar, limpiar la pantalla, posteriormente mostrar un menú tipo:

"Hola presione la tecla de la opción que desee:"

>1 Registrar usuario

>2 Acceder

>3 Salir

Si el usuario no presiona una tecla válida, le informan, esperan un enter para continuar, limpian pantalla y vuelven a imprimir el menú. Si no saben cómo hacer eso pueden practicar esto en otro proyecto hasta que lo logren, pueden usar try/catch, excepciones personalizadas, bucle while, recursión, etc.

Sigamos...

Si el cliente decide registrarse, deberán invocar a UserService desde el lado del Server, pues es el servidor el que conoce a la base de datos y el que va a registrar, tener en consideración lo siguiente:

1. El usuario debe escribir un usuario y contraseña válida, esas validaciones se las inventan ustedes. 
2. El usuario no debe existir en nombre y correo, de ser así notificar al cliente y que vuelva a intentarlo.
3. Si se registra con éxito, le informan, esperan un ENTER y le vuelven a mostrar el menú principal, o bien, lo redirigen a la pantalla de inicio de sesión (algo que todavía no han programado a hasta este punto)

Supongamos que el usuario tiene cuenta, entonces presiona la segunda opción, en ese caso deberán crear del lado del server algo tipo "LoggingService" con algún método que llama a UserService para recuperar los usuarios, ustedes deberán elegir cómo recuperarlos, ya sea por username, por email, etc. El punto es que una vez recuperado los campos seleccionados de logging deberán coincidir, recomiendo pedir username y password, recuperan por username y en LoggingService con el método logUser (que deberán crear) pueden devolver algún boolean, true si coinciden, false de no ser así, si es true, entonces ya puede entrar el usuario, de no ser así le mandan un error y le dan la opción de volver a intentarlo.

Una vez que los usuarios ya están felices y conviviendo en el chat, siempre que un usuario escribe algo (`in.readLine`) deberán mandar a llamar a MessageService y usar el método que hayan creado para guardar un mensaje en la base de datos.

Y creo es todo, ¡buena suerte!