# Base de datos

MongoDB se ejecuta como servicio independiente del backend y del frontend. El archivo `compose.yaml` inicia MongoDB 8 como replica set local, necesario para las transacciones que actualizan los saldos.

Desde la raíz del proyecto:

```powershell
docker compose -f database/compose.yaml up -d
docker compose -f database/compose.yaml ps -a
docker compose -f database/compose.yaml logs mongo-init
```

Espera a que `mongo-init` termine con código cero antes de iniciar el backend. La réplica anuncia `localhost:27017`, por lo que esta configuración está destinada al backend ejecutado en el mismo equipo. El backend usa `mongodb://localhost:27017/simulador?replicaSet=rs0` por defecto.

El volumen `mongo-data` conserva los datos entre reinicios. Para producción configura `MONGODB_URI` con una instancia remota que soporte replica sets y transacciones; no reutilices la configuración local sin revisar red, autenticación y secretos.
