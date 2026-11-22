# Base de datos

Este repositorio contiene la configuración para ejecutar MongoDB localmente. No contiene una copia de la base de datos de producción ni datos que deban cargarse en Atlas: Atlas administra esos datos como servicio. El backend y el frontend se mantienen en repositorios separados.

## MongoDB local con Docker

Desde la raíz de este repositorio:

```powershell
docker compose up -d
docker compose ps -a
docker compose logs mongo-init
```

Espera a que `mongo-init` termine con código cero antes de iniciar el backend. La réplica local anuncia `localhost:27017`; el backend usa por defecto `mongodb://localhost:27017/simulador?replicaSet=rs0`. El volumen `mongo-data` conserva los datos entre reinicios.

## Producción con MongoDB Atlas

No subas archivos de base de datos a GitHub. Para conectar el backend desplegado, guarda la cadena de conexión de Atlas en Render como variable secreta `MONGODB_URI`, con usuario, contraseña y nombre de base de datos. En Atlas, permite las conexiones de red del servicio de backend. Nunca publiques la cadena ni la contraseña en este repositorio.
