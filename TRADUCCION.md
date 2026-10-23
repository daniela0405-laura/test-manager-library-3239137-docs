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



## Evidencia por tarea (tasks.md)

| Tarea | Evidencia |
|---|---|
| T-01 | `GET /api/v1/health` responde `{"status":"ok"}` |
| T-02 | Migración `create_library_tables`: seis tablas, sin DEFAULT, cinco categorías con UUID fijos |
| T-09, T-10, T-11, T-16 | `php artisan test`: 10 pruebas en verde en `tests/Feature/LoanRulesTest.php`, cada una nombrada por su criterio (CA-01.3, CA-03.2, CA-06.2 a CA-06.5, CA-07.2, CA-10.4) |
| T-17 | `php artisan app:ensure-admin` crea el admin desde el entorno; con la base vacía se puede iniciar sesión |
| T-18 | Migración `add_domain_checks_t18`: un INSERT por `psql` con `total_copies = 0` falla con `ck_book_total_positive`; `data-model.md` §4 actualizado |
| T-21 | `php seed.php` en `-tool` crea 40 libros, 10 socios y 15 préstamos por la API; el reporte devuelve filas |
| T-22 | Parcial: login y catálogo paginado en React |
| T-23 | `index.html` estático en `-page` |

Las pruebas usan una base aparte (`library_test`) para no tocar los datos de demostración. Se crea con:

    docker exec library-db psql -U library -d library -c "CREATE DATABASE library_test;"
    docker exec library-db psql -U library -d library_test -c "CREATE SCHEMA IF NOT EXISTS library;"

Las pruebas encontraron un defecto real (la clase `BusinessException` no estaba en el repositorio) y se corrigió: así se usa la evidencia del artículo XII.

## Pendiente

- T-15 Portada (`/books/{id}/cover`).
- T-19 Pruebas de arquitectura: el contrato de `import-linter` es propio de Python; en Laravel el equivalente sería Pest Arch, no implementado.
- T-20 La API y el frontend aún no corren dentro de Docker; solo el motor.
- T-21 Los 3 préstamos vencidos no se pueden crear por la API sin un reloj inyectable (ver README de `-tool`).
- T-22 Pantallas de socios, préstamos, devolución, vencidos y reporte.



\- Portada (`/books/{id}/cover`).

\- Pantallas de React, sembrador en `tool` y pruebas automáticas.

