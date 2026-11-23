<!-- Sprint 1: del 23 al 30 de noviembre de 2026 (8 días). Total: 74 puntos, 137 horas. Estimaciones hechas a petición de Gianfranco; 1 punto equivale aproximadamente a 2 horas, redondeado a la serie de Fibonacci. -->

# Épica: Fundaciones del proyecto
- prioridad: crítica

Dejar listos los repositorios, el entorno de contenedores y las migraciones para que el resto del trabajo pueda avanzar sin bloqueos.

## Característica: Esqueletos de repositorios
- prioridad: crítica

Estructura inicial de cada repositorio según la decisión 011 (multirepositorio).

**Criterios de aceptación**
- [ ] Cada repositorio compila o abre sin errores con su estructura inicial
- [ ] Cada repositorio tiene un README breve

### Elemento del backlog: Esqueleto de vibra-backend
- prioridad: crítica
- puntos: 5
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero la solución .NET 11 con el host, los módulos y el worker para empezar a implementar funcionalidades.

**Criterios de aceptación**
- [ ] Existen `src/Vibra.Api`, `src/Vibra.Worker` y los siete módulos (Identity, Catalog, Library, Streaming, History, Search, Billing) con sus capas Domain, Application, Infrastructure y Api
- [ ] `dotnet build` termina sin errores
- [ ] Las pruebas de arquitectura (NetArchTest) verifican que un módulo no referencia la infraestructura de otro

#### Tarea: Crear la solución y el host `Vibra.Api`
- horas: 2
- sprint: Sprint 1

#### Tarea: Crear los siete módulos con sus cuatro capas
- horas: 3
- sprint: Sprint 1

#### Tarea: Crear el proyecto `Vibra.Worker`
- horas: 1
- sprint: Sprint 1

#### Tarea: Crear el proyecto de pruebas y las pruebas de arquitectura
- horas: 3
- sprint: Sprint 1

#### Tarea: Crear `scripts/export-openapi.sh`
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Base funcional de controladores y errores
- prioridad: alta
- puntos: 5
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero `FunctionalController`, `ResultHttpExtensions`, `IErrorHttpMapper<T>` y el ayudante base de Dapper para que todos los módulos usen el patrón Result de forma uniforme.

**Criterios de aceptación**
- [ ] Los errores se devuelven como Problem Details (RFC 7807) sin exponer `Exception.Message`, SQL ni trazas
- [ ] Los controladores solo pueden ser delgados: una expresión por acción
- [ ] Se admite `CancellationToken` en el ayudante de acceso a datos

<!-- Se portan desde rxlink-api con las correcciones indicadas en el CLAUDE.md del backend. -->

#### Tarea: Portar `FunctionalController` y `ResultHttpExtensions`
- horas: 3
- sprint: Sprint 1

#### Tarea: Portar `IErrorHttpMapper<T>` devolviendo `ProblemDetails`
- horas: 2
- sprint: Sprint 1

#### Tarea: Portar el ayudante base de Dapper con soporte de `CancellationToken`
- horas: 3
- sprint: Sprint 1

#### Tarea: Inyectar `TimeProvider` en lugar de usar `DateTime.UtcNow`
- horas: 1
- sprint: Sprint 1

#### Tarea: Pedir aprobación y añadir los paquetes NuGet pendientes (DbUp, NetArchTest)
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Esqueleto de vibra-ios
- prioridad: crítica
- puntos: 3
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero el proyecto de Xcode con los paquetes Swift locales para empezar a construir las funcionalidades.

**Criterios de aceptación**
- [ ] El objetivo mínimo de iOS es la última versión disponible en el Xcode instalado
- [ ] Existen los paquetes Domain, Networking, Persistence, Playback, Store, DesignSystem y Features
- [ ] Swift 6 con concurrencia estricta activada
- [ ] La app se ejecuta en el simulador

#### Tarea: Confirmar el número exacto de la última versión de iOS en el Xcode instalado
- horas: 1
- sprint: Sprint 1

#### Tarea: Crear el proyecto de Xcode y el objetivo delgado
- horas: 2
- sprint: Sprint 1

#### Tarea: Crear los paquetes Swift locales
- horas: 2
- sprint: Sprint 1

#### Tarea: Crear el archivo `Vibra.storekit` vacío
- horas: 1
- sprint: Sprint 1

#### Tarea: Crear `scripts/sync-contract.sh`
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Esqueleto de vibra-contracts
- prioridad: media
- puntos: 2
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero el repositorio del contrato OpenAPI para versionar la API compartida entre backend e iOS.

**Criterios de aceptación**
- [ ] Existen `openapi.json` inicial, `CHANGELOG.md` y `.spectral.yaml`

#### Tarea: Crear la estructura del repositorio con sus archivos iniciales
- horas: 2
- sprint: Sprint 1

#### Tarea: Definir la regla de etiquetado con versionado semántico
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Definir el alcance de vibra-db
- prioridad: media
- puntos: 1
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero decidir qué contiene `vibra-db` para no duplicar lo que ya viaja en las migraciones de cada módulo.

**Criterios de aceptación**
- [ ] El alcance queda registrado en la decisión 002 (solo SQL transversal, como la extensión `pg_trgm` y datos semilla, o ningún contenido)

#### Tarea: Decidir el alcance y registrarlo en la decisión 002
- horas: 1
- sprint: Sprint 1

## Característica: Entorno de contenedores con Podman
- prioridad: alta

Ejecutar el backend, PostgreSQL y Valkey sobre Podman (decisión 016).

**Criterios de aceptación**
- [ ] Un solo comando levanta PostgreSQL y Valkey en local

### Elemento del backlog: Levantar PostgreSQL y Valkey con Podman
- prioridad: alta
- puntos: 2
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero un archivo de composición para tener la base de datos y la caché locales.

**Criterios de aceptación**
- [ ] `podman compose up -d` inicia PostgreSQL y Valkey
- [ ] La API se conecta a ambos con secretos de usuario, sin credenciales en el repositorio

#### Tarea: Crear el archivo de composición
- horas: 2
- sprint: Sprint 1

#### Tarea: Configurar `dotnet user-secrets` para las cadenas de conexión
- horas: 1
- sprint: Sprint 1

#### Tarea: Documentar el arranque en el README del backend
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Actualizar el CLAUDE.md del backend
- prioridad: baja
- puntos: 1
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero que el `CLAUDE.md` del backend refleje Podman, DbUp y R2 para que no contradiga las decisiones.

**Criterios de aceptación**
- [ ] El archivo ya no menciona Docker, el emulador de blobs ni la herramienta de migraciones pendiente

#### Tarea: Aplicar los cambios redactados en el repositorio del backend
- horas: 1
- sprint: Sprint 1

## Característica: Migraciones con DbUp
- prioridad: alta

Migraciones SQL por módulo, con un diario por módulo (decisión 002).

**Criterios de aceptación**
- [ ] Cada módulo tiene su propio conjunto de migraciones y su propio diario

### Elemento del backlog: Ejecutar migraciones por módulo
- prioridad: alta
- puntos: 5
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como desarrollador quiero que las migraciones se apliquen en proceso al arrancar y en las pruebas con Testcontainers.

**Criterios de aceptación**
- [ ] Las migraciones de Identity y Catalog se aplican sobre una base vacía
- [ ] Volver a ejecutarlas no cambia nada

#### Tarea: Integrar DbUp en el host
- horas: 2
- sprint: Sprint 1

#### Tarea: Crear el mecanismo de diario por módulo
- horas: 3
- sprint: Sprint 1

#### Tarea: Crear las migraciones iniciales de Identity y Catalog
- horas: 3
- sprint: Sprint 1

# Épica: Verificaciones técnicas
- prioridad: crítica

Resolver cuanto antes los puntos "por verificar" que pueden cambiar el diseño.

## Característica: Spikes de riesgo
- prioridad: crítica

Cada spike termina con una conclusión escrita que actualiza la decisión correspondiente.

### Elemento del backlog: Spike: reproducir FLAC desde R2 con URL firmada
- prioridad: crítica
- puntos: 5
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Comprobar que `AVPlayer` reproduce y permite avanzar en un FLAC remoto servido por Cloudflare R2 y su CDN con acceso firmado y peticiones de rango. Es el punto de mayor riesgo del proyecto.

**Criterios de aceptación**
- [ ] `AVPlayer` reproduce y hace seek en un FLAC remoto en el simulador
- [ ] Queda definido cómo se combinan las URL firmadas con el CDN (las URL prefirmadas de R2 usan el punto de acceso S3, no un dominio personalizado)
- [ ] Quedan verificados los límites de la capa gratuita de R2 y las condiciones de Cloudflare para servir audio
- [ ] Se actualizan las decisiones 003 y 004

#### Tarea: Crear el bucket de R2 y subir un FLAC de prueba
- horas: 1
- sprint: Sprint 1

#### Tarea: Generar una URL firmada desde .NET
- horas: 2
- sprint: Sprint 1

#### Tarea: Probar la reproducción y el seek en el simulador
- horas: 2
- sprint: Sprint 1

#### Tarea: Probar el acceso a través del CDN
- horas: 3
- sprint: Sprint 1

#### Tarea: Registrar la conclusión
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Spike: versión de OpenAPI compatible con el generador de Swift
- prioridad: alta
- puntos: 2
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

ASP.NET Core 11 emite OpenAPI 3.2 por defecto. Comprobar qué versión admite `swift-openapi-generator` y configurar la exportación.

**Criterios de aceptación**
- [ ] `export-openapi.sh` genera un documento que el generador de Swift procesa sin errores
- [ ] Se actualiza la decisión 005

#### Tarea: Probar el generador de Swift con el documento exportado
- horas: 2
- sprint: Sprint 1

#### Tarea: Configurar la versión de OpenAPI de la exportación
- horas: 1
- sprint: Sprint 1

#### Tarea: Registrar la conclusión
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Spike: Testcontainers, Compose y CI con Podman
- prioridad: alta
- puntos: 5
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

**Criterios de aceptación**
- [ ] Las pruebas de integración con Testcontainers se ejecutan sobre Podman
- [ ] Queda decidido cómo se despliega en el VPS de Hetzner (Compose o Quadlet) y cómo se publica el contenedor en CI
- [ ] Se actualiza la decisión 016

#### Tarea: Ejecutar Testcontainers sobre Podman
- horas: 3
- sprint: Sprint 1

#### Tarea: Probar Compose o Quadlet en el VPS de Hetzner
- horas: 3
- sprint: Sprint 1

#### Tarea: Probar la publicación del contenedor en CI
- horas: 2
- sprint: Sprint 1

#### Tarea: Registrar la conclusión
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Spike: DbUp con .NET 11
- prioridad: media
- puntos: 2
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

**Criterios de aceptación**
- [ ] DbUp aplica una migración de prueba en .NET 11
- [ ] Se actualiza la decisión 002

#### Tarea: Aplicar una migración de prueba con DbUp en .NET 11
- horas: 2
- sprint: Sprint 1

#### Tarea: Registrar la conclusión
- horas: 1
- sprint: Sprint 1

### Elemento del backlog: Verificar el formato del CD ripeado
- prioridad: media
- puntos: 1
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

**Criterios de aceptación**
- [ ] Se confirma que los archivos son FLAC o WAV (los WAV se convierten con FFmpeg)
- [ ] Se actualiza la decisión 004

#### Tarea: Revisar el formato de los archivos y convertir con FFmpeg si es WAV
- horas: 2
- sprint: Sprint 1

# Épica: Reproductor, acceso y catálogo
- prioridad: alta

Esqueleto funcional: iniciar sesión, listar pistas y reproducir una en segundo plano. El reproductor es la pieza de mayor riesgo, por eso va primero.

## Característica: Inicio de sesión
- prioridad: alta

Registro e inicio de sesión con correo y contraseña (decisión 007).

**Criterios de aceptación**
- [ ] Un usuario puede registrarse e iniciar sesión desde la app
- [ ] Los tokens se guardan solo en el Keychain

### Elemento del backlog: API de identidad
- prioridad: alta
- puntos: 8
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como usuario quiero registrarme e iniciar sesión para acceder a la app.

**Criterios de aceptación**
- [ ] Existen los endpoints de registro, inicio de sesión y renovación bajo `/v1`
- [ ] El JWT es de vida corta y los tokens de renovación rotan
- [ ] Los errores esperados (credenciales inválidas, correo duplicado) se devuelven como Problem Details, sin lanzar excepciones

#### Tarea: Configurar ASP.NET Core Identity con Dapper
- horas: 4
- sprint: Sprint 1

#### Tarea: Endpoint de registro
- horas: 2
- sprint: Sprint 1

#### Tarea: Endpoint de inicio de sesión
- horas: 2
- sprint: Sprint 1

#### Tarea: Endpoint de renovación con rotación de tokens
- horas: 3
- sprint: Sprint 1

#### Tarea: Pruebas de integración con Testcontainers
- horas: 3
- sprint: Sprint 1

### Elemento del backlog: Pantallas de acceso en iOS
- prioridad: alta
- puntos: 5
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como usuario quiero pantallas en español de registro e inicio de sesión.

**Criterios de aceptación**
- [ ] Las cadenas de la interfaz están en español y usan `String(localized:)`
- [ ] Varias respuestas 401 simultáneas provocan una sola renovación de token

#### Tarea: Almacenar los tokens en el Keychain
- horas: 2
- sprint: Sprint 1

#### Tarea: Crear el actor de renovación de tokens
- horas: 3
- sprint: Sprint 1

#### Tarea: Pantallas de registro e inicio de sesión
- horas: 4
- sprint: Sprint 1

#### Tarea: Generar el cliente Swift a partir del contrato
- horas: 2
- sprint: Sprint 1

## Característica: Reproducción en segundo plano
- prioridad: crítica

Reproducir una pista con audio en segundo plano y controles en la pantalla de bloqueo (decisión 008).

**Criterios de aceptación**
- [ ] El audio sigue sonando con la app en segundo plano
- [ ] Los controles de la pantalla de bloqueo y del Centro de control funcionan

### Elemento del backlog: Servicio de reproducción
- prioridad: crítica
- puntos: 8
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como usuario quiero reproducir una pista y que siga sonando al salir de la app.

**Criterios de aceptación**
- [ ] `PlaybackService` vive fuera de las vistas y envuelve `AVPlayer`
- [ ] `AVAudioSession` usa la categoría `.playback` con el modo de fondo de audio
- [ ] Se gestionan interrupciones y cambios de ruta
- [ ] `MPNowPlayingInfoCenter` y `MPRemoteCommandCenter` muestran y controlan la pista

#### Tarea: Crear `PlaybackService`
- horas: 4
- sprint: Sprint 1

#### Tarea: Configurar la sesión de audio y el modo de fondo
- horas: 2
- sprint: Sprint 1

#### Tarea: Integrar `MPNowPlayingInfoCenter`
- horas: 2
- sprint: Sprint 1

#### Tarea: Integrar `MPRemoteCommandCenter`
- horas: 2
- sprint: Sprint 1

#### Tarea: Gestionar interrupciones y cambios de ruta
- horas: 3
- sprint: Sprint 1

### Elemento del backlog: Endpoint de reproducción con URL firmada
- prioridad: alta
- puntos: 3
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como usuario autenticado quiero obtener la URL de una pista para reproducirla.

**Criterios de aceptación**
- [ ] `GET /v1/tracks/{id}/stream` devuelve una URL firmada de corta duración
- [ ] Nunca se registran URL firmadas completas en los registros
- [ ] Los bytes de audio no pasan por la API

<!-- Todavía sin comprobación de derechos ni 403 con vista previa: eso es del Sprint 2 (módulo Billing). -->

#### Tarea: Implementar el módulo Streaming con generación de URL firmada
- horas: 4
- sprint: Sprint 1

#### Tarea: Pruebas de integración
- horas: 2
- sprint: Sprint 1

## Característica: Catálogo
- prioridad: alta

Listar artistas, álbumes y pistas desde la API.

**Criterios de aceptación**
- [ ] La app muestra una lista de pistas obtenida de la API

### Elemento del backlog: API de catálogo
- prioridad: alta
- puntos: 8
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como usuario quiero ver el catálogo de pistas, álbumes y artistas.

**Criterios de aceptación**
- [ ] Los endpoints de lectura usan paginación por cursor y ETags
- [ ] Cada pista y álbum incluye precio, moneda y `store_product_id`
- [ ] Los `operationId` son estables y descriptivos

<!-- Sin ingesta automática (Sprint 3): el catálogo inicial se carga de forma manual con datos semilla y archivos subidos a R2. -->

#### Tarea: Migraciones y repositorios de Catalog
- horas: 4
- sprint: Sprint 1

#### Tarea: Endpoints de lectura de pistas, álbumes y artistas
- horas: 4
- sprint: Sprint 1

#### Tarea: Datos semilla del catálogo
- horas: 2
- sprint: Sprint 1

#### Tarea: Subir manualmente los FLAC del CD a R2
- horas: 2
- sprint: Sprint 1

#### Tarea: Pruebas de integración
- horas: 3
- sprint: Sprint 1

### Elemento del backlog: Lista de pistas en iOS
- prioridad: alta
- puntos: 3
- sprint: Sprint 1
- asignado: Gianfranco Diaz Badoino

Como usuario quiero ver la lista de pistas y tocar una para reproducirla.

**Criterios de aceptación**
- [ ] La lista carga desde la API y permite reproducir una pista de principio a fin en el simulador

#### Tarea: Repositorio de catálogo en el paquete Domain
- horas: 2
- sprint: Sprint 1

#### Tarea: ViewModel y vista de la lista de pistas
- horas: 3
- sprint: Sprint 1

#### Tarea: Conectar la selección con `PlaybackService`
- horas: 2
- sprint: Sprint 1
