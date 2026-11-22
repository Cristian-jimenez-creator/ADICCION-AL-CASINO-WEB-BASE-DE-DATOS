# Base de datos

Este repositorio resuelve la configuración de MongoDB para el simulador. Contiene la definición para levantar una instancia local de desarrollo; **no contiene los datos de producción** ni un volcado que se deba cargar en Atlas. MongoDB Atlas aloja la base de datos de producción y administra su almacenamiento.

El backend es el único componente que se conecta a MongoDB. El frontend se comunica con la API del backend y nunca debe recibir la cadena `MONGODB_URI`.

## Tecnologías y archivos

- MongoDB 8.0.
- Docker Compose para el entorno local.
- Replica set local `rs0`, necesario para que el backend pueda realizar transacciones.
- Volumen Docker `mongo-data` para conservar los datos locales entre reinicios.
- `compose.yaml`: inicia MongoDB, espera su estado saludable e inicializa el replica set.

Este repositorio mantiene una configuración de ejecución, no un dump, colecciones exportadas ni un archivo de esquema de producción. Los documentos de cuentas y partidas se escriben desde el backend cuando se usa la aplicación.

No se mantiene un esquema SQL ni un conjunto de documentos de producción dentro de Git. El backend usa Spring Data MongoDB y crea/consulta los datos de la aplicación al ejecutarse.

## Iniciar MongoDB local

Necesitas Docker Desktop con Docker Compose. Desde la raíz de este repositorio:

```powershell
docker compose up -d
docker compose ps -a
docker compose logs mongo-init
```

Espera a que `mongo-init` finalice correctamente. El replica set se llama `rs0` y se anuncia como `localhost:27017`, por lo que esta configuración es para una aplicación ejecutada en el mismo equipo. El backend local usa:

```text
mongodb://localhost:27017/simulador?replicaSet=rs0
```

Para detener los contenedores sin borrar los datos:

```powershell
docker compose down
```

El volumen sigue existiendo. **No** uses `docker compose down -v` si deseas conservar los datos locales.

## Producción en MongoDB Atlas

1. Crea un cluster en Atlas y un usuario de base de datos con los permisos que necesita la aplicación.
2. Autoriza el acceso de red del servicio backend de Render en la lista de acceso IP de Atlas.
3. Copia la URI de conexión de Atlas y reemplaza los marcadores con los valores correctos; codifica los caracteres especiales de la contraseña para URL.
4. Guarda la URI completa como secreto `MONGODB_URI` en la configuración de Render. Incluye el nombre de base de datos y los parámetros que Atlas proporcione.
5. Reinicia o despliega el backend y revisa sus registros para confirmar la conexión.

No escribas la URI, la contraseña, IP privada ni datos de jugadores en este README, archivos de configuración públicos, capturas o commits. Las variables de producción se configuran en Render y los datos permanecen en Atlas; no hace falta subir archivos a este repositorio para que Atlas funcione.

## Relación entre los componentes

```text
Frontend (Vercel) ──HTTP/REST──> Backend (Render) ──MONGODB_URI──> MongoDB Atlas
```

En desarrollo local, el backend apunta a la instancia iniciada por este `compose.yaml`.

## Alcance y límites verificados

- compose.yaml levanta MongoDB 8.0 para desarrollo, anuncia el replica set local rs0, publica el puerto 27017, conserva archivos en el volumen mongo-data e inicializa el replica set mediante el servicio mongo-init.
- Esta composición local no configura usuario/contraseña de MongoDB. Úsala solo en un equipo de desarrollo confiable; no expongas ese puerto directamente a Internet.
- El repositorio no incluye un volcado ni datos iniciales de jugadores. Las colecciones y documentos de la aplicación se crean y actualizan cuando el backend ejecuta sus operaciones.
- Atlas es independiente de GitHub y Docker Compose: se crea/configura desde MongoDB Atlas y el backend accede mediante MONGODB_URI.

## Comprobaciones cuando el backend no conecta

1. En local, confirma que docker compose ps -a muestre MongoDB saludable y que mongo-init haya terminado correctamente.
2. Confirma que el backend local use mongodb://localhost:27017/simulador?replicaSet=rs0.
3. En Render, la URI debe ser la de Atlas, con el nombre de base de datos y la contraseña correcta (codifica caracteres reservados de URL). No uses localhost: allí significaría el propio contenedor de Render.
4. Comprueba que Atlas tenga un usuario de base de datos y una regla IP que permita conexiones desde Render; revisa los logs del backend sin publicar la URI.

El volumen mongo-data mantiene los datos locales tras docker compose down; docker compose down -v elimina ese volumen y sus datos.
