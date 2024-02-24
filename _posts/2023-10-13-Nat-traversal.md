---
title: Nat traversal
date: 2023-10-13 20:00:00 +0200
categories: [Nat, Traversal]
tags: [nat, traversal, udp, hole punching]
---
<style type="text/css">
 .post-content { text-align: justify; }
</style>

Hace unas semanas mi ISP activó [CG NAT](https://es.wikipedia.org/wiki/Carrier_Grade_NAT) (Carrier Grade Network Address Translation). Esto hizo que mis servidores dejasen de estar expuestos a internet con las reglas de port-forwarding que tenía configuradas.

Lejos de llamar a la operadora y pedir que la desactiven, decidí intentar saltármela. Si las apps P2P funcionan detrás de esta NAT por qué no iba a poder yo usar estas técnicas para mis servicios.
Así pues, me informé de como hacer NAT Traversal, UDP Hole Punching, STUN, TURN, ICE, etc. Además de servicios de "VPN p2p" como Tailscale, Zerotier... Estos resolvían mi problema, creando una red mesh entre mis dispositivos, y es lo que estuve usando.

Como me encantan los retos, me propuse implementar una solución yo mismo, a partir de los conocimientos adquiridos en las técnicas mencionadas.

## UDP Hole Punching

El "UDP Hole Punching" es una técnica usada para establecer una conexión directa entre dos dispositivos que están detrás de un firewall / NAT.

Nos aprovechamos de que los firewalls generalmente permiten las conexiones salientes, por lo que para atravesarlos iniciaremos la conexión en los dos dispositivos a la vez, haciendo creer al firewall que ambas partes están iniciando el intercambio de mensajes.

- Primero, el lado A inicia la conexión con un paquete UDP con destino B. Este es rechazado puesto que el firewall B no ha visto ningún paquete saliente con dirección A.

![Desktop View](/assets/img/nat-firewalls-5a.png)

- Segundo, el lado B inicia la conexión con un paquete UDP con destino A. Este sí llega correctamente puesto que A ha visto un paquete saliente con dirección B.

![Desktop View](/assets/img/nat-firewalls-5b.png)

- Finalmente, obtenemos una comunicación bidireccional.

![Desktop View](/assets/img/nat-firewalls-5c.png)

Hay varios puntos que hemos pasado por alto.

- ¿cómo sabe cada dispositivo cuál es la dirección y puerto del otro para así poder iniciar la conexión? Para ello tendremos un servidor de señalización, como el expuesto en el apartado [servidor de señalización](#servidor-de-señalización).
- ¿estas direcciones ip:puerto cuáles son? Ya que sabemos que la **NAT** las modifica. Para solucionar este problema hacemos uso de STUN, un protocolo que permite preguntar a un servidor público cuál es mi ip:puerto desde su punto de vista.

## Tipos de NAT

Para el Hole Punching distinguimos dos tipos de NAT:

- Endpoint Independent NAT (EIM): para una misma pareja de origen ip:puerto la traducción no depende del destino. Es decir, podemos preguntar al servidor STUN cuál es nuestra ip:puerto y ésta será la misma que vean nuestros peers.
- Endpoint Dependent NAT (EDM): para una misma pareja de origen ip:puerto, la traducción varía según el destino escogido, por lo que al preguntar al servidor STUN cuál es nuestra ip:puerto no coincidirá con lo que vean nuestros peers. Para identificar si nuestra NAT es de este tipo, preguntamos a varios servidores STUN desde el mismo socket, si las respuestas muestran diferente ip:puerto estamos en este caso.

## Casos

Según el tipo de NAT de cada parte, distinguimos 3 casos.

### 1. EIM-EIM

Si los dos dispositivos están en el caso EIM, no hay ningún problema, se registran en el servidor de señalización y podrán establecer la conexión puesto que las parejas ip:puerto son conocidas.

### 2. EIM-EDM

Si uno de los dos dispositivos están el el caso EDM, tendremos que tratarlo de una manera especial, ya que conocemos una pareja ip:puerto pero la otra va cambiando según el destino (el servidor STUN no nos puede ayudar). Haremos uso de **la paradoja del cumpleaños**.

Suponiendo que la ip que nos devuelve STUN en el lado EDM es correcta (REQ-2 [RFC 4787](https://datatracker.ietf.org/doc/html/rfc4787)), lo que no conocemos es el puerto. Este será 1 entre 65.535. Si podemos probar 100 puertos/segundo, en el peor caso serán 10 minutos escaneando puertos. Pero podemos hacerlo mejor, si en vez de abrir 1 puerto entre 65.535 en el lado EDM, abrimos 256 el número de peticiones aleatorias hasta encontrar una colisión se reduce mucho.

Las mates de esto se explican en [la paradoja del cumpleaños](https://en.wikipedia.org/wiki/Birthday_problem) y pueden ser comprobadas con esta [calculadora en python](https://github.com/danderson/nat-birthday-paradox).

| Número de pruebas | Probabilidad de éxito |
|:------------------|:----------------------|
|174                |50%                    |
|256                |64%                    |
|1024               |98%                    |
|2048               |99.9%                  |

Si podemos probar 100 puertos/segundo, en el peor de los casos tardaremos **20 segundos** en encontrar una colisión.

### 3. EDM-EDM

Podríamos aplicar el mismo truco que en el apartado anterior pero esta vez el espacio de búsqueda de colisión es mucho mayor por lo que con un escaneo de 100 puertos/segundo tardaríamos **28 minutos** en encontrar una colisión con un 99.9% de posibilidades. El problema reside en que los routers tienen una memoria finita, por ejemplo el Juniper SRX 300 puede almacenar un máximo de 64.000 sesiones activas. Habríamos consumido todas estas solo para iniciar una conexión, lo cual sería un completo **desastre**.

Debemos abandonar esta idea de probar y obtener la colisión por fuerza bruta. Si queremos establecer una conexión EDM-EDM podemos explorar técnicas más sofisticadas como analizar el patrón que sigue la NAT para intentar adivinar qué traducción de dirección va a realizar y así conseguir la colisión.

Generalmente, el no poder establecer una conexión EDM-EDM no es un problema porque las NAT de los routers de casa suelen ser EIM y quizás los de empresa sean EDM, por lo que los escenarios home-to-home, home-to-office y home-to-cloud están a salvo.

## Negociar varias NATs

¿Qué pasa si estás tras varias NATs? Este es mi caso puesto que estoy tras CG-NAT.

![Desktop View](/assets/img/nat-multiple-layers.png)

Realmente, da igual, cada capa de NAT hará su propia traducción y todo funcionará correctamente.

## Implementación

Todo esto lo he implementado en una librería de Python llamada [PyHolePuncher](https://github.com/enriqueesanchz/pyHolePuncher), para que a partir de ella se puedan establecer túneles UDP entre dos dispositivos que estén tras una NAT.

En el apartado ejemplos encontramos cómo utilizar la librería para crear un chat P2P. Primero se crea un usuario y se registra en el servidor de señalización, en este caso en [Rendezvous](https://github.com/enriqueesanchz/rendezvous). Cuando ambos peers estén registrados pueden unirse mediante un "Chat room". Con la opción "Chat to a peer" se crea el túnel UDP entre los dispositivos que permite intercambiar mensajes.

Actualmente soporta los siguientes casos:

- [x] EIM-EIM
- [ ] EIM-EDM
- [ ] EDM-EDM

### Servidor de señalización

Para este proyecto he creado un servidor de señalización de ejemplo, donde los peers pueden unirse a salas y compartir la información necesaria para establecer la conexión directa.

[Rendezvous](https://github.com/enriqueesanchz/rendezvous) se trata de un servicio REST simple, cuyos métodos están explicados en la documentación de Swagger que provee. Una vez los distintos peers estén registrados e intercambien la información, podemos pasar al UDP Hole Punching.

## Leer más

Para escribir este post me he inspirado en el blog de Tailscale, muy recomendado pues describe con precisión el fundamento tecnológico tras su producto, algo que sirve para aprender mucho sobre el tema. Si se quiere seguir profundizando:

- [How NAT Traversal works](https://tailscale.com/blog/how-nat-traversal-works/)
- [How Tailscale works](https://tailscale.com/blog/how-tailscale-works/)
