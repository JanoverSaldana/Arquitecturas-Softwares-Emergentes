# Arquitecturas-Softwares-Emergentes

### Pregunta 1 (2 puntos): Diagrama de Contexto (C4 Model - Nivel 1)

```
+--------------------------+           +--------------------------+
|  External Investor       |           |  Mining Company Admin    |
|  (Mobile/Web Interface)  |           |  (Internal System User)  |
+--------------------------+           +--------------------------+
             |                                 |
             |                                 |
             v                                 v
                   +--------------------------+
                   |      API Gateway         |
                   | (Unified Entry Point)    |
                   +--------------------------+
                                 |
        +------------------------+------------------------+
        |                        |                        |
+----------------+    +--------------------+    +---------------------+
| Investment Svc |    | Notification Svc   |    | Recommendation Svc  |
+----------------+    +--------------------+    +---------------------+
        |                        |                        |
        v                        v                        v
   +---------+            +------------+           +-----------------+
   | Stripe  |            |   Twilio   |           | AWS SageMaker   |
   +---------+            +------------+           +-----------------+
```

**Explicación**:

* Dos tipos de usuarios interactúan con la plataforma: inversores externos y administradores internos.
* API Gateway actúa como punto de entrada unificado hacia los microservicios.
* Cada microservicio se conecta con servicios externos específicos según su función.

---

### Pregunta 2 (6 puntos): Diagrama de Contenedores (C4 Model - Nivel 2) + Justificación

```
+-----------------------------------------------------+
|                    API Gateway                      |
|       (NGINX, Kong, etc. - Entrada unificada)       |
+-----------------------------------------------------+
                      |
       +--------------+--------------+--------------+
       |              |              |              |
+----------------+ +--------------------+ +---------------------+
| Investment Svc | | Notification Svc   | | Recommendation Svc  |
|  - CQRS        | |  - Twilio API      | |  - SageMaker SDK    |
|  - Stripe SDK  | |  - Event Pub       | |  - ML Inference     |
|  - Command Bus | |                    | |                     |
+----------------+ +--------------------+ +---------------------+
       |                    |                      |
+--------------+    +---------------+      +--------------------+
|  SQL DB      |    |  NoSQL DB     |      | SQL or NoSQL DB    |
+--------------+    +---------------+      +--------------------+

                    <-> Message Broker (Kafka/RabbitMQ) <->
```

**Justificación del diseño:**

* **Microservicios independientes** aseguran escalabilidad y facilidad de despliegue.
* **API Gateway** permite autenticación centralizada, routing, y throttling.
* **Balanceador de carga** trabaja en conjunto con el API Gateway o servicios como Kubernetes.
* **Message Broker** permite desacoplar servicios y mantener alta disponibilidad.
* **CQRS** separa claramente comandos de consultas, mejorando el rendimiento y la claridad.
* **Bases de datos por servicio** aseguran el principio de "Database per Service".
* **Servicios externos** cumplen con requerimientos funcionales (pagos, SMS, sugerencias).

---

### Pregunta 3 (12 puntos): Bounded Context "Investment"

#### Diagrama de Clases UML (Dominio)

```
+-----------------------------+
|        Investment          |  <<Aggregate Root>>
+-----------------------------+
| - investmentId: UUID       |
| - investorId: UUID         |
| - amount: Money            |
| - status: InvestmentStatus |
| - createdAt: DateTime      |
+-----------------------------+
| +registerInvestment()      |
| +confirmPayment()          |
| +cancelInvestment()        |
+-----------------------------+
             |
             | 1
             |------------+
                          | *
               +--------------------------+
               |     InvestmentEvent      | <<Entity>>
               +--------------------------+
               | - eventId: UUID          |
               | - type: EventType        |
               | - occurredOn: DateTime   |
               +--------------------------+

+----------------------+
|       Money          | <<Value Object>>
+----------------------+
| - value: Decimal     |
| - currency: String   |
+----------------------+
| +add(), +subtract()  |
+----------------------+

+------------------------------+
| RegisterInvestmentCommand    |
+------------------------------+
| - investorId: UUID           |
| - amount: Decimal            |
| - currency: String           |
+------------------------------+

+----------------------------------+
| RegisterInvestmentHandler       |
+----------------------------------+
| +handle(command): Investment     |
+----------------------------------+
```

**Explicación del diseño:**

* Se define el Aggregate Root `Investment`, que controla la consistencia del agregado.
* `InvestmentEvent` registra eventos importantes en el ciclo de vida.
* `Money` es un Value Object que asegura consistencia en cálculos monetarios.
* El patrón **Command Handler** centraliza la ejecución de comandos.

**Convenciones usadas:**

* Clases y enums: `UpperCamelCase`.
* Atributos y métodos: `lowerCamelCase`.
* Lenguaje: Inglés técnico por estándares de arquitectura moderna.

Este diseño está preparado para una implementación en arquitecturas modernas orientadas a dominio con alta escalabilidad.
