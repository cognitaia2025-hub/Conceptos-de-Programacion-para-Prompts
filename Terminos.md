### 1. Instrucciones "Atómicas" vs. "Arquitectónicas"

cuando pides cosas como: *"Hazme un servicio para el inventario"* o *"Crea una función para ver los gastos"*.

* **El error:** Cuando pides cosas por separado, la IA te resuelve el problema **en ese momento** con lo primero que encuentra en su base de datos.
* **El resultado:** En `inventory/service.py` usó `psycopg` con un pool manual, en `gastos/service.py` usó `psycopg2` de forma síncrona (un pecado en FastAPI), y en `db.py` te metió `asyncpg` y `databases`.
* **La lección:** No le pediste que respetara una **única fuente de verdad** o una arquitectura base. Le diste permiso de improvisar.

### 2. Ignorar el "Ciclo de Vida" de la Conexión

Si nunca le indicaste: *"Asegúrate de que todas las consultas a la base de datos se manejen mediante inyección de dependencias y context managers asíncronos"*.

* **El error:** Dejaste que la IA decidiera cómo abrir y cerrar la puerta del almacén. En el servicio de inventario, por ejemplo, el código tiene bloques `try...finally` pero no gestiona explícitamente el inicio y fin de la **transacción**.
* **El resultado:** Los logs chillan porque la IA escribió código que "devuelve la llave" pero deja la puerta entreabierta (`[INTRANS]`).

### 3. Falta de Restricciones Técnicas (Tech Stack)

Si no le dices a tu LLM ejemplo: *"Estamos usando FastAPI con PostgreSQL y quiero que todo el acceso a datos sea asíncrono usando ÚNICAMENTE psycopg 3"*, la IA va a mezclar versiones.

* **El error:** Tu `gastos/service.py` usa una clase con una conexión que se queda abierta en el atributo `self.conn`. Esto es código de hace 10 años, Salvador. En un entorno moderno de FastAPI, eso es veneno.

### 4. No pedir "Manejo de Transacciones"

Si solo le dices *"Guarda este dato"*, la IA hará un `INSERT`. Pero si no le dices *"Asegúrate de que cada operación sea transaccional y se cierre correctamente"*, omitirá el paso del `COMMIT` o el cierre del bloque de transacción.

* **El resultado:** El pool de conexiones tiene que hacer el trabajo sucio por ti haciendo un `rollback` automático cada vez que una función termina.

---

### ¿Cómo deberías haberlo pedido? (Para la próxima)

Para que no te vuelva a pasar, tus instrucciones deberían sonar así:

> *"Crea el servicio de [X], pero usa la conexión que ya tenemos definida en `db.py`. Usa obligatoriamente `async with` para el cursor y para la transacción, asegurando que siempre se cierre o se haga commit antes de retornar el resultado. No uses librerías nuevas, mantente fiel a Psycopg 3 asíncrono."*
