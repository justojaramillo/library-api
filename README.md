# Biblioteca · API y sitio web

Proyecto **Next.js 16** (App Router) que cumple dos papeles para la asignatura *Desarrollo Móvil* (Unidad 03):

1. **API REST** (`/api/...`) que consume la app móvil [`../library_app`](../library_app).
2. **Sitio web** con las mismas funcionalidades que la app (estudiante y bibliotecario).

Los datos y la autenticación están en **Firebase** (Authentication + Firestore), que solo se usa desde el servidor.

## Guías para construirla paso a paso

Orden de trabajo: **primero esta API, después la app.**

1. 📗 [`docs/Guia_API_Web_Biblioteca.pdf`](docs/Guia_API_Web_Biblioteca.pdf) — Guía 1: construye la API y el sitio web (semanas 9 a 12).
2. 📘 [`../library_app/docs/Guia_App_Movil_Biblioteca.pdf`](../library_app/docs/Guia_App_Movil_Biblioteca.pdf) — Guía 2: construye la app móvil.

Son tutoriales de construcción: cada archivo aparece completo y en orden. Se verifican con `../library_app/docs/replay.py`, que las reproduce en un directorio limpio y comprueba tipos, pruebas, lint y build (ver el README de la app).

> Usa `firebase-admin` **13** a propósito: la 14 depende de `jwks-rsa` 4, que hace `require()` de un módulo ESM y falla en Vercel si su Node es anterior a 22.12 (500 sin mensaje, `ERR_REQUIRE_ESM`).

## Puesta en marcha

1. **Firebase** (consola): crea un proyecto, habilita *Authentication → Correo/contraseña*, crea una base de datos *Firestore* y genera una clave de cuenta de servicio (detalle en la sección 2.2 de la guía).
2. Copia `.env.example` a `.env` y completa las cuatro variables.
3. Instala y carga datos:

```bash
pnpm install
pnpm seed          # carga los 10 libros
pnpm dev           # http://localhost:3000   (pnpm dev:lan para usarla desde el teléfono)
```

**Usuarios:** no hay cuentas ni contraseñas escritas en el repositorio. Crea las tuyas (la contraseña queda solo en tu terminal):

```bash
pnpm create-user -- estudiante@tucorreo.com "TuClave" "Tu Nombre" student
pnpm create-user -- bibliotecario@tucorreo.com "TuClave" "Tu Nombre" librarian
```

> ⚠️ El `.json` de la cuenta de servicio y `.env` son credenciales: no se suben a git (ya están en `.gitignore`). Mueve o borra el `.json` después de copiar sus valores. 

## Comandos

| Comando | Qué hace |
|---|---|
| `pnpm dev` / `pnpm dev:lan` | Desarrollo (el segundo acepta conexiones de la red local) |
| `pnpm build` / `pnpm start` | Producción |
| `pnpm test` | Pruebas unitarias (Vitest) |
| `pnpm typecheck` · `pnpm lint` | Tipos y reglas de código |
| `pnpm seed` | Carga los 10 libros (no crea usuarios) |
| `pnpm create-user -- correo clave "Nombre" librarian` | Crea (o actualiza) un usuario con rol `librarian` o `student` |
| `pnpm set-role -- correo librarian` | Cambia el rol de un usuario que ya existe |
| `pnpm smoke` | Prueba de humo contra Firebase real (41 comprobaciones; usa usuarios temporales y limpia todo lo que crea) |
| `pnpm reset-demo` | Borra préstamos y portadas y restaura ejemplares |

## Endpoints

| Ruta | Quién |
|---|---|
| `POST /api/auth/register` · `login` · `refresh`, `GET /api/auth/me` | Público / autenticado |
| `GET /api/books` (`?q=&category=&updatedSince=`), `GET /api/books/:id` | Autenticado |
| `GET /api/books/:id/cover` | Público |
| `PUT /api/books/:id/cover` | Bibliotecario |
| `GET /api/loans`, `POST /api/loans` (idempotente por `clientRequestId`) | Autenticado / estudiante |
| `PATCH /api/loans/:id` (`approve` · `reject` · `return`) | Bibliotecario |

Errores uniformes: `{ "error": { "code": "NO_COPIES", "message": "…" } }`.

## Estructura

```text
app/api/        Route Handlers (la API)
app/(auth)/     login y registro del sitio web
app/(app)/      libros y préstamos del sitio web
app/actions.ts  Server Actions
lib/domain/     reglas de préstamo (puras, con pruebas)
lib/data/       acceso a Firestore
lib/services/   Firebase Authentication
proxy.ts        redirección optimista a /login
scripts/        seed, set-role, smoke, reset-demo
tests/          pruebas con Vitest
docs/           guía en Markdown y PDF
```

Notas de Next.js 16 que usa el proyecto: `proxy.ts` (antes *middleware*), `cacheComponents` (lo que depende de la petición se lee dentro de `<Suspense>`) y tipos `RouteContext`/`PageProps` generados con `next typegen`.
