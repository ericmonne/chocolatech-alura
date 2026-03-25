# **HR Buddy — Guion Completo Detallado (Revisado)**

Chatbot de RR. HH. con RAG, MySQL, OpenAI y Telegram, orquestado por n8n.

3 clases de 45 minutos cada una.

## **Preparación General para el Instructor (Pre-Clase)**

* **Lista de verificación:**  
  * \[ \] Ejecutar el *Load Data Flow* (carga del manual de RR. HH.) cada vez que se reinicie n8n. La base de datos en memoria (Simple Vector Store) se borra cuando la máquina virtual se reinicia.  
  * \[ \] Verificar si MySQL en Railway está Online (los planes gratuitos se suspenden por inactividad).  
  * \[ \] Tener Telegram web/móvil abierto para la demostración final.  
  * \[ \] Garantizar que las credenciales de OpenAI/Gemini (para Chat y Embeddings) y Telegram Bot estén activas.

## **CLASE 01 — El Cerebro de HR Buddy (RAG, Chunking y n8n)**

### **📌 Objetivo**

Crear dos flujos (pipelines) esenciales: uno para leer el PDF (Carga) y otro para responder preguntas basadas en ese PDF (Conversación usando Vector Store).

### **Conceptos Abordados**

| Concepto | Analogía / Explicación |
| :---- | :---- |
| RAG | Darle un libro a la IA para que lo lea antes de responder |
| Chunking | Por qué no enviar el documento entero (límites de tokens y costo) |
| Embeddings | Transformar texto en vectores para búsqueda por similitud |
| n8n | Herramienta low-code visual, ideal para cursos y prototipado |
| n8n Cloud vs. local | Cloud es gestionado y más fácil para empezar; local te da más control |

### **Flujo 1: Carga de Datos (Load Data Flow)**

Resumen: Extrae el archivo TXT/PDF de GitHub, lo "rebana" en pedazos y lo guarda en la memoria del Agente.

\<img width="1093" height="479" alt="image" src="https://github.com/user-attachments/assets/fab4ad27-54b3-401e-8bd1-382f7dcdd604" /\>

**Paso a Paso de los Nodos: (resumen)**

1. **Manual Trigger:** Inicia el proceso manualmente.  
2. **HTTP Request:** Extrae el manual de la empresa.  
   * *Method:* GET  
   * *URL:* https://raw.githubusercontent.com/ericmonne/chocolatech-alura/refs/heads/main/Manual%20de%20RH%20ChocolaTech%20LAD.txt 
   * *Response Format:* Text (en Options → Response. Por defecto n8n intenta leer JSON y falla).  
3. **Simple Vector Store (Insert):** Guarda los vectores en la memoria.  
   * Conecta al conector "Document": **Default Data Loader** (para fraccionar el texto automáticamente).  
   * Conecta al conector "Embedding": **Embeddings OpenAI** usando el modelo text-embedding-3-small.  
   * Operation: Retrieve Documents (As Tool for AI Agent)  
   * *Memory Key:* vector\_store\_key.  
   * Description: Busca información sobre políticas de RR. HH. de la empresa ChocolaTech  
4. **Embeddings OpenAI (ambos nodos):**  
   * Modelo: text-embedding-3-small  
   * Atención: los dos nodos de embeddings (Insert y Retrieve) DEBEN usar el mismo modelo.  
5. **Default Data Loader:**  
   * Type of Data: JSON  
   * Text Splitting: Simple  
   * Resultado: generó 22 chunks a partir del documento.

**Paso a paso detallado para el Flujo de Carga:**

### **Paso 1: El Disparador Manual**

1. Haz clic con el botón derecho en la pantalla (o en el "+") y haz clic en **Add node**.  
2. Busca **Manual Trigger** (en la versión en español puede aparecer como "Al hacer clic en 'ejecutar'").  
3. Funciona como un botón de "Play" para iniciar este proceso, ya que solo necesitará ejecutarse **una única vez** cada vez que lo vayas a usar.

### **Paso 2: La Descarga del Documento**

1. Tira de la línea del conector derecho del Manual Trigger.  
2. Agrega y conecta un nodo llamado **HTTP Request**.  
3. **Configuración Crucial del nodo HTTP Request:**  
   * **Method:** GET  
   * **URL:** Pega exactamente la ruta de GitHub enseñada en el curso:  
     https://raw.githubusercontent.com/ericmonne/chocolatech-alura/refs/heads/main/Manual%20de%20RH%20ChocolaTech%20LAD.txt
   * ⚠️ **ATENCIÓN:** Justo abajo, necesitas ir a la sección **Options**, hacer clic en **Add Option**, seleccionar **Response** y luego **Response Format**. Cambia la casilla format de *JSON* (por defecto) a **Text**. Sin esto, el nodo fallará diciendo que no pudo interpretar el documento.

### **Paso 3: Creando la Memoria Vectorial (La Inserción)**

1. Tira de la línea de salida principal del HTTP Request.  
2. Agrega un nodo llamado **Simple Vector Store**.  
3. **Configuración del Vector Store:**  
   * Nota que es exactamente igual al de la clase anterior, pero con un rol diferente.  
   * **Operation:** Déjalo como **Insert Documents** (insertar / guardar datos).  
   * **Memory Key:** Escribe exactamente **vector\_store\_key**. Es a través de esta clave que el Agente del flujo 2 sabrá dónde se escondieron las políticas.

### **Paso 4: El "Fraccionador" (El Data Loader)**

Aquí es donde preparamos el documento gigante en pequeños pedacitos legibles (Chunking).

1. Mira la base de tu nuevo Simple Vector Store.  
2. Tira de una línea del subconector derecho, llamado **"Document"**.  
3. Agrega un nodo llamado **Default Data Loader**.  
4. **Configuración del Data Loader:**  
   * **Type of Data:** Marca JSON (sí, aunque sea un archivo de texto, el campo de datos generado por el nodo anterior exige este formato).  
   * **Text Splitting:** Selecciona Simple. (Esto dividirá automáticamente el texto bruto en unos 22 bloques perfectos para que la IA los procese).

### **Paso 5: El Modelador Matemático (Embeddings)**

Aquí vamos a transformar el texto fraccionado en coordenadas que los robots entienden (*vectores*).

1. Vuelve a la base del Simple Vector Store.  
2. Tira de una línea paralela a partir del subconector izquierdo, llamado **"Embedding"**.  
3. Agrega el nodo **Embeddings OpenAI**.  
4. **Configuración del Embedding:**  
   * **Credential:** Agrega tu clave de OpenAI.  
   * **Model:** Asegúrate de llenar con **text-embedding-3-small**. Esto es obligatorio.

### **Cómo Ejecutar**

Tu lienzo (canvas) ahora tiene dos "dibujos" aislados.

Para ejecutar este Flujo de Carga:

1. Posiciona el mouse encima del Manual Trigger recién creado.  
2. Aparecerá el clásico ícono de "Play" (Test step). Presiónalo.  
3. Si todo se pone verde (Success), eso significa que el documento bajó de GitHub, fue leído, fraccionado y guardado en el "Cerebro" de tu n8n.  
4. ¡Ahora puedes ir a la Ventana de Chat y decir un "Hola" al Agente para probar la información\!

### **Flujo 2: Conversación Base Inicial (Retriever Flow)**

Resumen: Ventana de chat nativa de n8n que responde en base a lo que guardamos en el paso de carga.

\<img width="834" height="610" alt="image" src="https://github.com/user-attachments/assets/f8f599a2-c217-43da-ae18-253718743ec2" /\>

**Paso a Paso de los Nodos: (resumen)**

*(Atención: Este flujo presupone que ya ejecutaste el **Flujo 1 (Carga)** al menos una vez para que el texto de las políticas ya esté guardado en la memoria vectorial bajo la clave vector\_store\_key).*

1. **When chat message received:** Disparador de chat nativo de n8n (usaremos esto solo hasta la Clase 02).  
2. **AI Agent:** Cerebro principal.  
   * *Source for Prompt:* Connected Chat Trigger Node  
   * Conecta por debajo (Model): **OpenAI Chat Model** (ej: gpt-4o-mini).  
   * Conecta por debajo (Memory): **Simple Memory** (solo uno).  
   * Conecta por debajo (Tool): **Simple Vector Store**.  
     * *Operation:* Retrieve Documents (As Tool for AI Agent).  
     * *Consejo Crítico:* Conecta a él exactamente el **mismo nodo Embeddings OpenAI** (text-embedding-3-small) usado en la carga. ¡Mezclar Google Gemini y OpenAI destruye la brújula vectorial del Agente\!

**Paso a paso detallado:**

### **Paso 1: El Punto de Entrada (Disparador)**

1. Haz clic en el botón de **"+"** (Add node) en la pantalla vacía de n8n.  
2. Busca **When chat message received** (también llamado "Chat Trigger") y agrégalo.  
3. Este nodo no necesita configuraciones adicionales. Crea esa pantalla de chat azul en la parte inferior de n8n para que puedas probar.

### **Paso 2: El Cerebro Principal**

1. Haz clic en el pequeño conector derecho del nodo "When chat message received" y arrastra la línea.  
2. En la lista que aparece, busca y agrega el nodo **AI Agent**.  
3. **Configuración del nodo AI Agent:**  
   * **Source For Prompt:** Déjalo como Connected Chat Trigger Node (esto hará que el texto que escribas en el chat vaya directo al agente).  
   * *Opcional:* Si quieres, puedes llenar el campo System Message diciendo cómo debe actuar, por ejemplo: *"Eres el HR Buddy, asistente de RR. HH. de ChocolaTech. Responde solo sobre políticas de la empresa."*

### **Paso 3: El Modelo de Lenguaje**

1. El nodo AI Agent tiene pequeños conectores en su base. Haz clic y arrastra el conector marcado como **"Model"** (o "Chat Model").  
2. Busca y agrega el nodo **OpenAI Chat Model**.  
   * *(Nota: Si tu cuenta de OpenAI no tiene créditos, como vimos anteriormente, puedes buscar Google Gemini Chat Model o Groq Chat Model aquí).*  
3. **Configuración del Modelo:**  
   * **Credential:** Selecciona tu cuenta de API (de OpenAI, Google, etc).  
   * **Model:** Si es OpenAI, selecciona el gpt-4o-mini (es el más rápido y barato recomendado en el curso).

### **Paso 4: La Memoria de la Conversación**

1. Vuelve al nodo AI Agent y arrastra una línea a partir del conector **"Memory"**.  
2. Agrega **solo un** nodo llamado **Simple Memory** (o Window Buffer Memory dependiendo de la versión de tu n8n).  
3. **Configuración de la Memoria:**  
   * **Session ID:** Déjalo como Connected Chat Trigger Node (esto separa el historial por sesión de chat).  
   * Nuevamente, ¡asegúrate de tener **solo un** nodo de memoria conectado a tu agente\!

### **Paso 5: El Puente con las Políticas de la Empresa (Herramienta Vector Store)**

Aquí es donde ocurre la magia del RAG. Vamos a darle el manual de la empresa al chatbot para que lo lea.

1. En el nodo AI Agent, arrastra una línea a partir del conector llamado **"Tool"**.  
2. Agrega el nodo **Simple Vector Store**.  
3. **Configuración Crucial del Vector Store:**  
   * **Operation:** Es obligatorio cambiar de Insert a **Retrieve Documents (As Tool for AI Agent)**. Si pones solo Retrieve o Insert, no funcionará.  
   * **Memory Key:** Escribe exactamente la misma clave que usaste en el flujo de carga, que según el guion es vector\_store\_key.  
   * **Tool Description:** Este campo le informa al modelo cuándo debe usar esta herramienta. Escribe algo como: *"Busca información sobre políticas de RR. HH., vacaciones, horarios y rutinas de la empresa ChocolaTech"*.

### **Paso 6: El Traductor Vectorial (Subnodo de Embeddings)**

1. El nodo Simple Vector Store que acabas de agregar tiene un subconector debajo llamado **"Embedding"**.  
2. Arrastra una línea de ese conector y agrega el nodo **Embeddings OpenAI**.  
3. **Configuración del Embedding:**  
   * **Model:** Obligatoriamente usa **text-embedding-3-small** (o exactamente el mismo que usaste en el flujo de carga anterior, de lo contrario no se encontrarán los textos originales).

Errores e Inconsistencias Encontrados

4. ### **La plantilla RAG creó un workflow duplicado: Al usar el "RAG starter template" de n8n, se creó un workflow con ID duplicado causando errores en bucle.**

   * **Solución**: Archivar el workflow problemático.  
5. **Read/Write Files from Disk no funciona en n8n Cloud**: El nodo lee archivos del servidor de n8n, no de la máquina local.  
   * **Solución**: Usar HTTP Request para buscar el archivo directamente desde GitHub.  
6. **HTTP Request devolvía JSON en lugar de Text**: El campo Response Format no aparecía visualmente, pero estaba disponible en Options → Response.  
   * **Solución**: Cambiar la opción a Text.  
7. **Text Splitter no es un nodo independiente**: En el n8n actual, el chunking se hace dentro del Default Data Loader, no como un nodo separado.  
8. **Simple Vector Store "Retrieve" vs. "Retrieve for AI Agent"**: Son opciones diferentes dentro del mismo nodo.  
   * **Solución**: Para usarlo como Tool del AI Agent, seleccionar Retrieve Documents for AI Agent as Tool.  
9. **Se necesitan dos nodos de Embeddings OpenAI**: Uno para el flujo de inserción (Load Data Flow) y otro para el flujo de recuperación (Retriever Flow). Ambos DEBEN usar el mismo modelo.

### **Resultados de las Pruebas**

* Pregunta: "¿A cuántos días de vacaciones tengo derecho?" → Respuesta correcta basada en el documento ✅  
* Pregunta fuera de alcance (ej.: capital de Francia) → El bot también respondió ⚠️ (las barreras o guardrails se introducen en la Clase 03\)

## **CLASE 02 — Memoria Corporativa (Base de Datos MySQL)**

\<img width="1049" height="628" alt="image" src="https://github.com/user-attachments/assets/930dc76b-a760-41e5-84eb-dd85e85425a8" /\>
```sql
CREATE TABLE empleados (  
    id INT AUTO_INCREMENT PRIMARY KEY,  
    nombre VARCHAR(100) NOT NULL,  
    email VARCHAR(150) NOT NULL UNIQUE,  
    departamento VARCHAR(100) NOT NULL,  
    puesto VARCHAR(100) NOT NULL,  
    fecha_ingreso DATE NOT NULL,  
    saldo_vacaciones INT NOT NULL DEFAULT 0,  
    banco_horas DECIMAL(5,1) NOT NULL DEFAULT 0,  
    modalidad VARCHAR(20) NOT NULL DEFAULT 'hibrido'  
);

INSERT INTO empleados (nombre, email, departamento, puesto, fecha_ingreso, saldo_vacaciones, banco_horas, modalidad) VALUES  
('Juan Silva', 'juan.silva@empresa.com', 'Ingeniería', 'Ingeniero de Software', '2022-03-10', 20, 0.0, 'hibrido'),  
('María Souza', 'maria.souza@empresa.com', 'Recursos Humanos', 'Analista de RR. HH.', '2021-05-15', 5, 12.5, 'hibrido'),  
('Carlos Oliveira', 'carlos.oliveira@empresa.com', 'Finanzas', 'Analista Financiero', '2023-01-20', 0, 0.0, 'presencial'),  
('Ana Lima', 'ana.lima@empresa.com', 'Marketing', 'Especialista en Marketing', '2020-11-05', 15, -4.0, 'remoto'),  
('Pedro Santos', 'pedro.santos@empresa.com', 'Ventas', 'Ejecutivo de Ventas', '2022-08-01', 10, 8.0, 'hibrido'),  
('Fernanda Costa', 'fernanda.costa@empresa.com', 'Operaciones', 'Gerente de Operaciones', '2019-02-12', 30, 0.0, 'presencial'),  
('Rafael Mendes', 'rafael.mendes@empresa.com', 'TI', 'Analista de Soporte', '2023-06-10', 0, 15.5, 'hibrido'),  
('Juliana Rocha', 'juliana.rocha@empresa.com', 'Ingeniería', 'Desarrolladora Front-end', '2021-09-25', 12, 0.0, 'remoto'),  
('Bruno Alves', 'bruno.alves@empresa.com', 'Diseño', 'Diseñador UX/UI', '2022-04-18', 8, 3.5, 'hibrido'),  
('Camila Ferreira', 'camila.ferreira@empresa.com', 'Atención al Cliente', 'Analista de Atención al Cliente', '2024-01-05', 0, 0.0, 'hibrido'),  
('Eric Monné', 'eric.monne@chocolatech.com', 'Producto', 'Instructor de Cursos', '2024-01-15', 25, 8.0, 'hibrido');
```
### **📌 Objetivo**

Dar identificación al empleado. El robot buscará en la base de datos MySQL (Railway) los datos de quien está conversando.

### **Conceptos Abordados**

| Concepto | Explicación |
| :---- | :---- |
| RAG vs. SQL | La búsqueda vectorial encuentra reglas y políticas; MySQL encuentra datos específicos del empleado |
| Por qué ambos | El PDF no sabe quién es João; MySQL no conoce las reglas de vacaciones |
| System Prompt con datos inyectados | Cómo pasar contexto dinámico al AI Agent |
| $fromAI() | Función de n8n para que el AI Agent complete parámetros de herramientas dinámicamente |

### **Qué se Agregó al AI Agent**

* Nueva Tool: **MySQL** (Select rows from funcionarios)  
* System Message actualizado con la lógica de identificación del empleado

### **Paso a Paso de los Nodos (resumen)**

1. Mantenemos el Retriver Flow de la Clase 01\.

### **Concepto 1: ¿Por qué RAG y SQL juntos?**

* **RAG (Búsqueda Vectorial en la Clase 01):** Es excelente para leer textos no estructurados y reglas (ej: *"¿Cómo funciona el banco de horas?"*). Sin embargo, el PDF del manual no tiene la menor idea de quién es "Ana Lima".  
* **SQL (Búsqueda Estructurada en la Clase 02):** Es excelente para datos matemáticos y exactos de personas específicas (ej: *"Ana tiene un saldo de 15.5 horas en el banco"*). Sin embargo, MySQL no entiende las reglas subjetivas de la empresa.  
  **La magia de nuestro Agente:** Al juntar las dos herramientas bajo el mismo robot, si Ana pregunta *"¿Puedo tomarme el día libre mañana usando mi banco de horas?"*, el Agente irá a MySQL a leer que ella tiene 15.5 horas, y luego irá al Vector Store a leer que la política exige al menos 8 horas para cambiarlas por un día libre. ¡Cruza ambas informaciones y responde a su pregunta a la perfección\!  
2. A partir del conector "Tool" del **AI Agent**, tira de una segunda línea y agrega el nodo **Select rows from a table in MySQL**.  
3. **Configuración de la Credencial MySQL (Railway):**  
   * Activa el "Public Networking / TCP Proxy Domain" en el panel de Railway para acceso externo.  
   * *Host:* caboose.proxy.rlwy.net (ejemplo) | *Puerto:* 23446 (ejemplo)  
   * *Usuario:* root | *Base de datos:* railway  
4. **Configuración de la Búsqueda (Nodo MySQL):**  
   * *Table:* funcionarios | *Operation:* Select  
   * En **Select Rows \-\> Conditions**, configura:  
     * *Column:* nome  
     * *Operator:* Cambia a **Like** (si dejas Equal, la búsqueda falla por pequeños desencuentros de espaciado o apellidos).  
     * *Value:* Haz clic en el ícono de las estrellas mágicas **(✨)** al lado derecho del campo para activar el autocompletado por la IA vía $fromAI().  
       * *Name:* nome  
       * *Description:* "Solo el nombre del empleado, envuelto entre porcentajes para búsqueda parcial. Ejemplo: %Ana Lima%"

### **System Prompt del AI Agent (El Secreto)**

En el nodo AI Agent, actualiza la caja **System Message** con reglas para darle libertad al robot de buscar exactamente lo que el usuario escribe:

Eres el HR Buddy, asistente virtual de RR. HH. de ChocolaTech.  
REGLAS:  
1\. Responde siempre en español.  
2\. Responde SOLO dudas relacionadas con RR. HH.  
IDENTIFICACIÓN DEL EMPLEADO:  
\- Si el usuario no dice quién es, pregúntale su nombre completo en la primera respuesta.  
\- Usa la herramienta MySQL para buscar en la tabla funcionarios usando SIEMPRE el NOMBRE COMPLETO informado por el usuario en la conversación.  
\- Si se encuentra: usa los saldos de vacaciones y banco de horas.  
\- Si no se encuentra: no inventes datos personales. Responde solo con base en las políticas generales de RR. HH. del Vector Store.  
Usa la base de conocimientos para dudas generales.

**Paso a paso detallado**

### **Paso 1: Agregando la Herramienta MySQL al Agente**

Usarás el mismo flujo de chat que construiste en la Clase 01\.

1. Ve a tu nodo **AI Agent**.  
2. En la parte inferior del nodo, donde está el conector llamado **"Tool"** (herramienta), tira de una nueva línea y suéltala en un espacio vacío. *(Sí, tendrás dos herramientas conectadas al mismo botón Tool: el Vector Store y ahora la BD).*  
3. Busca y agrega el nodo **Select rows from a table in MySQL**.

### **Paso 2: Configurando la Conexión a la Base de Datos en la Nube**

1. Haz doble clic en el nodo MySQL recién creado.  
2. En la parte superior, en **Credential**, agrega una nueva conexión para MySQL.  
3. **Concepto de Public Networking:** Tu base de datos está alojada en Railway (un proveedor en la nube). n8n necesita acceder a ella "desde afuera". Por lo tanto, debes ir al panel de Railway, habilitar el botón de red pública (Public Networking/TCP Proxy Domain) y tomar la dirección (Host) y el Puerto generados allí.  
4. Llena la credencial con el Host, el Puerto, el Usuario (root), la Contraseña y la Base de datos (railway). Guarda la credencial.

### **Paso 3: Creando las "Condiciones" y Explicando el $fromAI()**

Con el nodo MySQL abierto, vamos a enseñarle cómo hará la búsqueda.

1. **Operation:** Selecciona Select (queremos leer datos, no borrarlos ni insertarlos).  
2. **Table:** Escribe funcionarios (la tabla que creamos con el script SQL).  
3. En la sección principal, agrega una **Condition** (Condición de búsqueda):  
   * El nombre de la columna (Field) es nome.  
   * El operador (Operator) debe ser **Like** *(Concepto: usar "Like" previene que la base de datos dé error en caso de que la IA pase espacios en blanco sin querer).*  
4. **El secreto de la IA: El comando $fromAI()**  
   * En el campo de Valor (Value), no vas a escribir un nombre fijo. ¡La base de datos necesita buscar dinámicamente el nombre de la persona con la que está conversando\!  
   * Pasa el mouse sobre el campo de valor y haz clic en el ícono de las estrellas (✨) que es el botón **"From AI"**.  
   * Nombra el parámetro como: nome.  
   * En la **Descripción del parámetro**, escribe una regla estricta: *"Usa siempre el formato %nombre% (con % al inicio y al final) para búsqueda parcial. Ejemplo: %João Silva%"*.  
   * **Concepto:** La función {{ $fromAI(...) }} es un puente mágico de n8n. Crea un formulario invisible donde la IA, al leer el chat, "adivina" qué nombre escribió el usuario y llena ese espacio vacío automáticamente antes de ir a la base de datos.

### **Paso 4: El "System Prompt" (Dándole Identidad al Robot)**

Ahora que la herramienta está lista, necesitamos dar órdenes explícitas al "cerebro" principal del Agente para que sepa cómo usar estos datos nuevos.

1. Haz doble clic en el nodo principal, el **AI Agent**.  
2. Ve a la caja de texto grande llamada **System Message** (o Instrucciones del Sistema).  
3. Escribe (con tus propias palabras, pero manteniendo la lógica del guion) cómo debe actuar. Ejemplo:  
   "Eres el HR Buddy, el asistente de RR. HH. de ChocolaTech. Responde SOLO dudas sobre RR. HH., reglas y beneficios.  
   IDENTIFICACIÓN:  
   Al inicio de cada conversación, pregunta:  
   '¡Hola\! Soy HR Buddy. ¿Cuál es tu nombre completo?'  
   Con el nombre proporcionado, USA LA HERRAMIENTA MYSQL para buscar los datos de esa persona en la tabla.  
   Usa el saldo de vacaciones, banco de horas, régimen y cargo para darle respuestas personalizadas.  
* Si el empleado es encontrado: usa los datos devueltos (saldo\_ferias, banco\_horas, regime, cargo, departamento) para personalizar las respuestas.  
* Si el empleado NO es encontrado: informa 'No encontré tu registro en el sistema. Puedo responder solo dudas generales sobre las políticas de RR. HH.'  
  Usa la base de conocimientos para responder dudas sobre políticas y reglas de RR. HH."

### **¡Listo\! Prueba la Clase 02**

* Prueba con Pedro Santos: el bot preguntó el nombre, buscó en MySQL y respondió con datos personalizados ✅  
* Prueba con nombre inexistente: el bot informó que solo puede responder dudas generales ✅  
* Combinación RAG \+ MySQL: "¿Puedo tomarme vacaciones ahora? ¿Cómo hago para solicitarlas?" — la respuesta usó las políticas del documento Y los datos de la base de datos ✅

### **Errores e Inconsistencias Encontrados**

1. **MySQL interno vs. externo en Railway**: El host mysql.railway.internal solo funciona dentro de la red de Railway. n8n Cloud no tiene acceso a esta red.  
   * **Solución**: Habilitar Public Networking en Railway y usar el host público (caboose.proxy.rlwy.net:23446).  
2. **Advertencia de versión en MySQL Workbench**: Railway usa MySQL 9.4, y Workbench fue homologado hasta la versión 8.0.  
   * **Solución**: Hacer clic en "Continue Anyway" para continuar normalmente.  
3. **Error "No database selected" en Workbench**: Ocurre cuando el schema no está seleccionado en la barra lateral (sidebar).  
   * **Solución**: Hacer doble clic en el schema railway en la barra lateral antes de ejecutar el script.  
4. **Railway no tiene un editor SQL nativo funcional**: El campo de query en la pestaña Database/Data sirve solo para visualización.  
   * **Solución**: Usar MySQL Workbench para ejecutar scripts.  
5. **$fromAI() devuelve undefined en la vista previa de n8n**: Comportamiento esperado en tiempo de diseño.  
   * En ejecución real, el AI Agent completa el valor correctamente.  
6. **Timeout de MySQL al cargar columnas en n8n**: En algunos momentos Railway presenta lentitud al cargar metadatos.  
   * **Solución**: Escribir el nombre de la columna manualmente en lugar de esperar la carga automática.

### **⚠️ Error Clásico de la Clase 02**

En el nodo MySQL, la alerta roja *"No parameters are set up to be filled by AI"* avisa que la herramienta es inútil. Este aviso MATA al Agente, obligándolo a intentar adivinar la tabla de otras formas y entrar en un bucle de Query. Solo desaparece cuando el alumno hace clic en el aviso mágico **(✨)** en el campo Value, activando correctamente la función $fromAI().

## **CLASE 03 — Producto Real (Telegram y Guardrails)**

\<img width="1385" height="757" alt="image" src="https://github.com/user-attachments/assets/e5958031-1646-477b-aa02-cdb714147ba3" /\>

### **📌 Objetivo**

1. Transformar el proyecto en una aplicación real en el celular.  
2. Crear una "Barrera" (Guardrails) bloqueando preguntas de curiosos (como recetas de pasteles) ANTES de que activen el Agente de IA, ahorrando dinero.  
3. Resolver el problema de la Memoria Sin Estado (Stateless) enseñando el Session ID aislado por persona.

### **Conceptos Abordados**

| Concepto | Explicación |
| :---- | :---- |
| Webhook | Puerta de entrada pública que transforma n8n en una API REST |
| Guardrails | Filtro que clasifica y bloquea preguntas fuera del alcance del chatbot |
| Stateless | El Webhook no tiene memoria nativa entre llamadas |
| Chat Trigger vs. Webhook | Chat Trigger es conversacional (tiene sesión); Webhook es API (sin sesión) |
| Telegram como interfaz | Gratuito, integración vía BotFather, y el chat.id resuelve el problema de sesión |
| Embeddings | Transformar texto en vectores numéricos para búsqueda por similitud |
| Default Data Loader | Preparar y fragmentar datos para el Vector Store |

### **Conceptos Cruciales de la Clase 03**

1. **Telegram vs Chat Trigger (El problema de la Memoria Local):**  
   En las clases anteriores, usaste la ventana azul de chat del propio n8n. Esa ventana tiene una "sesión" automática. Cuando pasas al mundo real usando **Webhooks** o aplicaciones como **Telegram**, cada mensaje que llega es tratado como un evento único y aislado (concepto llamado *Stateless* / Sin Estado).  
   **La Solución de Telegram:** Telegram proporciona un número único para cada usuario que habla con el robot, llamado chat.id. ¡Vamos a inyectar este ID en la memoria de n8n para que el Agente sepa quién es quién en el historial\!  
2. **Guardrails (Las Barreras de Protección):**  
   Los modelos de lenguaje son caros. Si un empleado comienza a pedir recetas de pasteles o códigos de programación a tu bot de RR. HH., vas a gastar dinero de la API en vano. El Guardrail es una barrera estructural: colocaremos un "clasificador" barato y rápido justo en la puerta de entrada. Si la pregunta es sobre RR. HH., se activa el Agente principal. Si está fuera de contexto, el flujo se corta antes incluso de activar el cerebro principal.

### **Por Qué Telegram Resuelve el Problema de Memoria**

## **El Webhook no tiene sesión nativa. Telegram proporciona un chat.id único y persistente para cada conversación. Este ID se usa como session key en el nodo Simple Memory, garantizando que el bot recuerde el contexto de la conversación de cada usuario.**

### **Paso a Paso de los Nodos (resumen)**

*Consejo para la Clase: Abandona el antiguo flujo de la Clase 02 y crea un **Workflow Nuevo** "HR Buddy \- Telegram" para no ensuciar la didáctica.*

1. **Telegram Trigger (La Puerta de Entrada):**  
   * Agrega y crea las credenciales (Token rescatado con @BotFather). *Updates:* message.  
2. **Message a Model (El Clasificador / Portero):**  
   * Conecta al Telegram Trigger. Usa un Model OpenAI puro y barato (gpt-4o-mini).  
   * *Prompt:* "Clasifica el siguiente mensaje en tres bloques: SALUDO, RRHH u OTRO. Mensaje: {{ $json.message.text }}. Responde solo con UNA PALABRA."  
3. **Switch (El Filtro/Barrera):**  
   * Enruta el camino dependiendo de la palabra clasificada:  
     * *Rule 1 (RRHH):* Si {{ $json.output\[0\].content\[0\].text }} es igual a RRHH.  
     * *Rule 2 (SALUDO):* Si es igual a SALUDO.  
     * *Fallback (OTRO):* En la sección Options (abajo), define "Fallback Output" como Extra Branch.  
4. **Respuestas Directas (Para Saludo y Fallback):**  
   * Agrega nodos **Send a text message (Telegram)** en las salidas secundarias.  
   * Textos: *"¡Hola, soy el asistente de RR. HH\!"* y *"Lo siento, solo respondo cuestiones de RR. HH."*.  
   * **ID de Telegram:** Código: {{ $('Telegram Trigger').item.json.message.chat.id }}. Enséñalo mediante el método visual de arrastrar (ver consejo al final).  
5. **La Ruta Principal de RR. HH. (El AI Agent Completo):**  
   * En la salida "RRHH" del Switch, reconstruye tu **AI Agent** (usando las herramientas de la Clase 02: Vector Store Mapeado y MySQL Tool).  
   * Como ya no está conectado al Chat Nativo azul:  
     * *Source for Prompt:* Define below.  
     * *Prompt:* Escribe {{ $('Telegram Trigger').item.json.message.text }} *O arrastra la palabra text del nodo Telegram Trigger usando el historial.*  
   * En el **Simple Memory**, configura la separación del historial por usuario:  
     * *Session ID:* Define below  
     * *Key:* Escribe el id del chat de Telegram: {{ $('Telegram Trigger').item.json.message.chat.id }}.  
6. **Envío Final:**  
   * Cierra la punta verde del AI Agent conectándolo a un último nodo **Send a text message**.  
   * *Chat ID:* El mismo id rescatado.  
   * *Text:* Escribe literalmente {{ $json.output }} (Donde reposa el mensaje elaborado y súper inteligente generado por el Agente final).

### **Paso a Paso detallado**

#### **Paso 1: Configurando Telegram (El Disparador)**

Lo primero es crear el bot en la aplicación de Telegram.

1. Abre tu Telegram y busca el robot oficial llamado **@BotFather**.  
2. Envía el comando /newbot, elige el nombre de visualización de tu bot (ej: "HR Buddy ChocolaTech") y luego un nombre de usuario que termine en bot (ej: chocola\_hr\_bot).  
3. BotFather te devolverá un "Token" gigante. Cópialo.  
4. En n8n (en el lienzo vacío), agrega el nodo **Telegram Trigger**.  
5. Crea una nueva Credencial (Telegram API) pegando ese Token.  
6. En la configuración del disparador, selecciona **Updates:** message (para que reaccione solo cuando alguien envíe un mensaje real de texto).

#### **Paso 2: El Clasificador (Guardrail)**

Antes de que el mensaje llegue al Agente caro, vamos a ahorrar tokens y tiempo haciendo una clasificación tonta, pero eficiente.

1. Tira de la línea del Telegram Trigger y agrega un nodo de LLM puro, llamado **Message a Model** (herramienta base de OpenAI).  
2. Conecta tu credencial de OpenAI y elige el modelo gpt-4o-mini.  
3. En el prompt, escribe una instrucción exacta:*"Clasifica el siguiente mensaje en una de las tres categorías: SALUDO / RRHH / OTRO.*  
   *Mensaje: {{ $json.message.text }}*  
   *Responde SOLO con una palabra única."*

#### **Paso 3: El Enrutador / Filtro (El Nodo Switch)**

Ahora, vamos a hacer un desvío en las vías dependiendo de la palabra que soltó el Clasificador.

1. Tira de la línea del clasificador y agrega el nodo **Switch**.  
2. **Agrega las reglas de enrutamiento (Rules):**  
   * **Regla 1 (RRHH):** Prueba si el output del nodo anterior (generalmente en $json.output\[0\].content\[0\].text o $json.message.content dependiendo del nodo) contiene la palabra RRHH.  
   * **Regla 2 (Saludo):** Prueba si el output contiene la palabra SALUDO.  
   * **Fallback (Extra):** El Switch tiene una salida por defecto para "todo lo demás". Esto se encargará de los temas de la categoría "OTRO".

#### **Paso 4: Respuestas Directas (Para Saludo y Fallback)**

1. En las salidas del nodo Switch referentes a **SALUDO** y al **Fallback**, agrega un nodo **Telegram** común (con la acción Send a text message).  
2. **Configuración Exigida en TODOS los nodos de envío de Telegram:**  
   * **Chat ID:** Como el Switch borró los datos originales del camino, debes forzar a n8n a ir a buscar el ID inicial: {{ $('Telegram Trigger').item.json.message.chat.id }}  
   * **Text (Saludo):** *"¡Hola\! Soy el asistente de RR. HH. de ChocolaTech..."*  
   * **Text (Fallback):** *"Lo siento, solo fui entrenado para responder dudas relacionadas con los RR. HH. de esta empresa."*  
   * Fallback: opción de Extra Branch (o Extra Output / Return items that don't match any rules)

#### **Paso 5: El Cerebro Real (Solo para Preguntas de RR. HH.)**

Si la palabra clasificada fue "RRHH", ¡entonces sí activamos el flujo pesado que construimos en la Clase 02\!

1. En la salida de la Regra 1 (RRHH) del Switch, agrega y construye todo tu nodo **AI Agent** exactamente igual al de la clase pasada (conectando el *Chat Model*, la herramienta *Vector Store* y la herramienta *MySQL*).  
2. Tira de la salida verde del AI Agent y envíala a **un último nodo Telegram** (Send a text message), usando el mismo Chat ID forzado del paso anterior y definiendo el campo de texto (Text) como la respuesta bruta generada por el Agente Inteligente ({{ $json.response }} o equivalente).  
3. En el campo Chat ID, pasa el mouse del lado derecho y haz clic en el botón de "expresión" (el símbolo de engranaje) y luego en Add Expression (Agregar Expresión). Nodes \-\> Telegram Trigger \-\> Output Data \-\> Json \-\> Message \-\> Chat \-\> Id

#### **Paso 6: ¡El Secreto de la Memoria en Telegram\!**

¡Cuidado\! Como el Agente de IA ya no está conectado directamente a un Chat Trigger nativo (está en medio del flujo), necesitamos configurarlo manualmente:

1. En el nodo **Simple Memory** conectado al Agente, cambia el campo *Session ID* de *Connected Node* a **Define below** (Definir a continuación).  
2. En la pequeña llave que aparece, coloca el ID de la conversación de la aplicación de la persona, para usarlo como número único de la memoria: {{ $('Telegram Trigger').item.json.message.chat.id }}.  
3. En el propio nodo del **AI Agent**, cambia la opción *Source for Prompt* a **Define below** y en el campo Prompt que surge, fuerza el llenado con el texto original de la persona usando {{ $('Telegram Trigger').item.json.message.text }}.

### **7\. Campo "Chat ID" (¿A quién le enviará el bot?)**

¿Recuerdas ese camino de "pesca" que enseñé antes? Aquí es **exactamente igual**.

* Abre el editor de Expresiones (en el cuadrito del lado derecho de Chat ID).  
* En el panel izquierdo, expande: **Nodes** \> **Telegram Trigger** \> **Output Data** \> **JSON** \> **message** \> **chat**.  
* Arrastra la variable **id** a la caja central.  
* *Código generado:* {{ $('Telegram Trigger').item.json.message.chat.id }}  
  *(¡Siempre buscamos desde el inicio para garantizar que el mensaje llegue al celular de la persona correcta\!)*

### **8\. Campo "Text" (¿Qué va a decir el bot?)**

Aquí, necesitas pasar adelante la respuesta inteligentísima que el AI Agent acaba de inventar en el nodo anterior. Como el AI Agent es el nodo que está pegado (conectado a la entrada) de este nodo de Telegram, es mucho más fácil.

El *Código generado es algo súper corto:* {{ $json.output }}.

### **Errores e Inconsistencias Encontrados**

1. **Simple Memory falla sin el session ID correcto con Telegram**: El nodo espera el session ID del Chat Trigger nativo. Con Telegram Trigger, es necesario configurarlo manualmente.  
   * **Solución**: Usar $('Telegram Trigger').item.json.message.chat.id como Key en el nodo Simple Memory.  
2. **chat\_id está vacío en los nodos Send a text message**: Después del Switch, el $json ya no carga la referencia a message.chat.id.  
   * **Solución**: Referenciar siempre el Telegram Trigger directamente: $('Telegram Trigger').item.json.message.chat.id.  
3. **El Output del clasificador OpenAI tiene una estructura diferente a la esperada**: Al usar el nodo OpenAI (no el AI Agent), el output viene en $json.output\[0\].content\[0\].text, no en $json.message.content.  
   * **Solución**: Ajustar la ruta de la expresión en el Switch.  
4. **Respond to Webhook no interpreta expresiones dentro de JSON**: Al usar JSON con {{ $json.output }}, n8n devolvía la expresión como texto literal.  
   * **Solución**: Usar la opción First Incoming Item en el nodo de respuesta.  
5. **Simple Memory incompatible con n8n Cloud Queue Mode**: Los datos de sesión se pierden si n8n se reinicia.  
   * **Para producción**: Usar Redis o PostgreSQL como backend de memoria.  
6. **Simple Vector Store pierde datos al reiniciar**: Por ser in-memory, el índice vectorial se borra cuando n8n se reinicia.  
   * **Solución para el curso**: Ejecutar siempre el Load Data Flow antes de iniciar las pruebas. Comunicar esta limitación a los alumnos.  
7. **Workflow archivado de la versión Webhook genérico**: El workflow original Prepare HR Buddy \- Class 3 (basado en Webhook genérico) fue reemplazado por la versión con Telegram y archivado.  
8. **AI Agent "No prompt specified" con Telegram Trigger**: El AI Agent configurado como "Connected Chat Trigger Node" busca un campo chatInput que solo existe en el Chat Trigger nativo — no en el Telegram Trigger.  
   * **Solución**: Cambiar "Source for Prompt" a Define below y definir el Prompt como {{ $('Telegram Trigger').item.json.message.text }}.  
9. **MySQL LIKE sin comodines hace match exacto**: Al usar el operador Like en el nodo MySQL Tool, pasar solo el nombre sin % equivale a una coincidencia exacta — si el agente pasa "João Silva" sin %, la consulta falla para cualquier variación.  
   * **Solución**: En el campo Description del Value del nodo MySQL Tool, instruir al modelo: *"Usa siempre el formato %nombre% (con % al inicio y al final) para búsqueda parcial. Ejemplo: %João Silva%"*  
10. **Empleado no encontrado incluso con el nombre correcto**: Si el nombre no está registrado en la tabla funcionarios, la consulta devuelve vacío y el bot responde solo con políticas generales.  
    * **Solución**: Insertar al empleado en la base de datos con el script a continuación y usar búsqueda parcial con LIKE.

### **Agregando un Nuevo Empleado a la Base de Datos**

Para agregar un empleado manualmente (ej.: para demostración en clase), ejecuta en MySQL Workbench:

USE hr\_buddy;

INSERT INTO funcionarios (nome, email, departamento, cargo, data\_admissao, saldo\_ferias, banco\_horas, regime) VALUES  
('Christian Velasco', 'christian.velasco@techsolutions.com.br', 'Produto', 'Instrutor de Cursos', '2024-01-15', 25, 8.0, 'hibrido');

### **💡 Consejo de Oro Didáctico (La Técnica de la "Cinta Transportadora")**

Cuando enseñes a los alumnos cómo rescatar el Chat ID y el mensaje original usando expresiones llenas de puntuación ($().item.json...), evita enfocarte en el código y muestra el **n8n visual**:

**Guion para hablar:** *"Chicos, el Switch reemplazó nuestro texto por la palabra 'RRHH', ¡así que el ID del usuario original se perdió\! Como n8n funciona como una cinta transportadora, vamos a 'tirar de un hilo' desde el inicio. Usen el Engranaje (Add Expression), abran el panel izquierdo: Nodes \-\> Telegram Trigger \-\> Output Data \-\> JSON \-\> message \-\> chat. ¡Hagan clic en id y simplemente arrástrenlo a la caja\! ¡Vean la magia generando el código feo por sí sola\!"*

## **Notas Generales para el Instructor**

### **Lista de verificación Antes de Cada Clase**

* \[ \] Ejecutar el Load Data Flow (Manual Trigger → HTTP Request → Simple Vector Store Insert) para garantizar que el documento esté cargado en la memoria  
* \[ \] Verificar si el servicio MySQL en Railway está online (el plan gratuito puede suspenderse por inactividad)  
* \[ \] Tener Telegram abierto en el celular o en el escritorio para demostración en vivo  
* \[ \] Confirmar que las credenciales OpenAI y MySQL estén activas en n8n

### **Limitaciones Importantes para Mencionar en Clase**

| Limitación | Impacto | Alternativa para Producción |
| :---- | :---- | :---- |
| Simple Vector Store es in-memory | Los datos desaparecen si n8n se reinicia | Pinecone, Qdrant, pgvector |
| Plan gratuito de Railway | Latencia o suspensión por inactividad | Plan pago de Railway u otro proveedor |
| Plan gratuito de n8n Cloud | Límite de 50 ejecuciones por mes | Plan pago o n8n self-hosted |
| Simple Memory en n8n Cloud | Datos perdidos al reiniciar | Redis o PostgreSQL |
| OpenAI tiene un costo | text-embedding-3-small y gpt-4o-mini son los más baratos, pero cobran por token | Monitorear el uso en el panel de OpenAI |

### **Repositorio del Manual de RR. HH.**

* Repositorio: https://github.com/ericmonne/chocolatech-alura  
* URL raw del archivo: https://raw.githubusercontent.com/ericmonne/chocolatech-alura/main/Manual%20de%20RH%20ChocolaTech.txt
