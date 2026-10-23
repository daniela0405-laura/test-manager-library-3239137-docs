\# Traducción del spec a React + Laravel



Se respeta el negocio del spec (historias, criterios, reglas, contrato y modelo de datos). Se traduce solo la tecnología.



\## Mapa de traducción



| Decisión en el spec (Python / .NET) | Traducción en este reto |

|---|---|

| Framework web (FastAPI / ASP.NET Core) | Laravel 13: `routes/api.php` + Controllers |

| Validación (Pydantic / DataAnnotations) | `$request->validate()` de Laravel |

| ORM (SQLAlchemy / EF Core) | Eloquent |

| Migraciones (Alembic / EF Migrations) | Migraciones de Laravel |

| Motor PostgreSQL 16, esquema `library` | `postgres:16` en Docker; `search\_path=library` |

| Token JWT de 8 h | Laravel Sanctum con expiración de 8 h (decisión D-1) |

| Hash de contraseña | `Hash::make` (bcrypt) |

| Cuerpo de error único (RNF-4) | Un solo manejador en `bootstrap/app.php` |

| Autorización por rol | Middleware `EnsureRole` |

| Frontend | React + Vite |



\## Modelo de datos



\- Seis tablas en singular: cada modelo declara `$table`.

\- Ids UUID v4 con `HasVersion4Uuids` (el `HasUuids` de Laravel genera v7).

\- Sin `created\_at`/`updated\_at` ni `DEFAULT`.

\- `ck\_book\_available\_range` como CHECK en la base (RN-1, CA-07.3).

\- ISBN único solo entre activos: índice parcial `WHERE deleted\_at IS NULL` (CA-03.2).

\- Baja lógica con `SoftDeletes` (CA-03.5).

\- FKs RESTRICT, salvo `loan\_item.loan\_id` con CASCADE; `category\_name` sin FK a propósito.

\- Las cinco categorías se siembran en la migración con sus UUID fijos.



\## Reglas de negocio



\- Estado del préstamo calculado; no existe columna `status` (CA-08.3).

\- Préstamo en una transacción con descuento condicional `available\_copies > 0`: con dos peticiones por el último ejemplar, una gana y la otra recibe 409 (RNF-2).

\- La línea congela título, ISBN y categoría (RN-4); el reporte agrupa por esos valores (CA-10.4).

\- Devolución con UPDATE condicional `returned\_at IS NULL`: nunca suma dos veces (CA-07.2).

\- Duplicados responden 409 con su código (`ISBN\_TAKEN`, `USERNAME\_TAKEN`, `DOCUMENT\_TAKEN`), no el 422 por defecto de Laravel.

\- El admin se crea con `php artisan app:ensure-admin` desde `ADMIN\_USERNAME` y `ADMIN\_PASSWORD`.



\## Decisiones



\- \*\*D-1 Sanctum en lugar de JWT.\*\* Token opaco nativo de Laravel con expiración de 8 h. La respuesta del login respeta el contrato. Agrega la tabla técnica `personal\_access\_tokens`.

\- \*\*D-2 Sesión y caché en archivo.\*\* Se quitaron las tablas `cache`, `jobs` y `sessions` para no agregar tablas que el modelo no tiene.



\## Pendiente



\- Portada (`/books/{id}/cover`).

\- Pantallas de React, sembrador en `tool` y pruebas automáticas.

