# 1. Actualizar el parámetro size al valor correcto de 384 acorde a bge-small-en-v1.5

Ubicación: README.MD / Sección 3. Crear una colección (client.create_collection / VectorParams(size=4, ...)).

Se configuró estáticamente un tamaño de vector de size=4 en la creación de la colección de Qdrant. Sin embargo, en la sección anterior se definió el modelo de embeddings BAAI/bge-small-en-v1.5, el cual genera de forma nativa representaciones vectoriales de 384 dimensiones. Intentar almacenar estos embeddings generará un error crítico de tamaño en Qdrant (Status 400: Vector size mismatch)
Cambiar a:

```
from qdrant_client.models import Distance, VectorParams

client.create_collection(
    collection_name="Nombre",
    vectors_config=VectorParams(size=384, distance=Distance.DOT),
)
```
# 2. Ruptura de la Arquitectura Parent-Child en el Pipeline ETL (Posible Incoherencia Funcional)

Ubicación: README.MD / Función procesar_skill_md(ruta_archivo).

En la sección conceptual del proyecto se estipula la estrategia de recuperación Parent-Child / Small-to-Big, donde cada hijo vectorizado debe conservar un identificador o el contenido del bloque padre para entregarlo al LLM. No obstante, en la implementación de la función procesar_skill_md, el bucle que procesa los hijos simplemente los acumula en docs_para_qdrant perdiendo la traza y el contenido del bloque padre (parent), lo cual invalida la lógica RAG descrita.

# 3. Exposición de Servicios y Ausencia de Autenticación en Qdrant (Vulnerabilidad de Seguridad)

Ubicación: README.MD / Sección Qdrant (Comando docker run y subsección Security Fix).

El comando expone los puertos por defecto (6333 y 6334) vinculándose a todas las interfaces de red (0.0.0.0) sin requerir credenciales ni tokens de autenticación (api-key). Si el entorno de desarrollo se ejecuta en una máquina con exposición a redes locales o se utiliza algún túnel de pruebas, cualquier cliente no autorizado podrá leer o modificar la colección de reglas lógicas.
Es necesario habilitar autenticación por API Key en Qdrant o restringir el acceso únicamente a la interfaz local (Preferiblemente esta).

# 4. Manejo Inseguro de Claves de API en Clientes Locales (Antipatrón de Conectividad)

Ubicación: README.MD / Sección 1. Servidor de LM Studio apuntando a modelo de razonamiento (ChatOpenAI).

Pasar explícitamente api_key="" puede provocar fallos de validación en versiones recientes de las librerías de integración si el esquema espera un string no vacío u obligatorio, a pesar de apuntar a un servidor local (http://localhost:1234/v1).
Buscar una versión específica de langchain-openai que permita cadenas vacías sin lanzar excepciones de validación en tiempo de ejecución.
