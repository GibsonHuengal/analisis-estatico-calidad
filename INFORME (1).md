# Informe de análisis estático automático

**Curso:** Evolución y Configuración de Software – Semana 11 (Taller de gestión del cambio)
**Fecha del análisis:** 30/10/2026
**Herramientas:** ESLint + `eslint-plugin-sonarjs` (reglas de code smells de SonarSource) + `eslint-plugin-security`, `jscpd` (duplicación), `npm audit` (dependencias), compilador TypeScript y Jest (cobertura).
**Criterio de aprobación:** equivalente a la puerta de calidad *Sonar way* mostrada en clase (sin bugs, sin vulnerabilidades, cobertura ≥ 80 %, duplicación ≤ 3 %).

> Nota de transparencia: este informe se generó con las herramientas anteriores, que miden las mismas dimensiones que SonarQube. El análisis con el servidor SonarQube real se ejecuta con `scripts/sonarqube-local.sh` (ver README); en el entorno donde se preparó este repositorio no se pudo levantar Docker ni descargar SonarQube, por lo que esos resultados **no** están incluidos aquí.

---

## 1. Proyecto descargado de GitHub: `ualmtorres/nestjs-espacios` (v0.0.1)

API REST en NestJS + TypeORM + SQLite para gestionar espacios, reservas y solicitudes. 760 líneas de código (sin tests).

| Condición de calidad | Valor medido | Estado |
|---|---|---|
| Compila sin errores | sin errores | APROBADO |
| Pruebas unitarias | 13 pasan de 13 | APROBADO |
| Cobertura ≥ 80 % | **39,01 %** líneas, 0 % ramas | FALLA |
| Duplicación ≤ 3 % | **12,33 %** (7 bloques) | FALLA |
| Vulnerabilidades en dependencias | **13 críticas, 35 altas** (87 en total) | FALLA |
| Problemas de seguridad en código | 0 hallazgos | APROBADO |
| Code smells | **13 hallazgos** | FALLA |

**Resultado: puerta de calidad NO APROBADA (4 de 7 condiciones fallan).**

### 1.1 Mantenibilidad
- **Duplicación (12,33 %)**: los bloques repetidos están en `solicitud-repository-mock.ts`, `solicitud-service-mock.ts` y los dos `*.spec.ts` de `solicitud` (entre 13 y 24 líneas repetidas). Recomendación: centralizar datos de prueba en un único *fixture* compartido.
- **Code smells (13)**: 7 variables/imports sin usar (`MiddlewareConsumer` en `app.module.ts`, `Reserva` en `create-espacio.dto.ts`, parámetros en los mocks, `of` en el spec) y 6 literales de texto repetidos 3 veces en los tests (`sonarjs/no-duplicate-string`).
- Ninguna función supera el umbral de complejidad cognitiva (15).

### 1.2 Seguridad
- **Dependencias**: 13 críticas y 35 altas. Las críticas directas son `@nestjs/cli`, `class-validator` y `typeorm` (versiones antiguas del proyecto: Nest 8, TypeORM 0.2). Recomendación: actualizar a versiones vigentes y volver a ejecutar `npm audit`.
- **Hotspots a revisar** (no los detecta una regla automática, pero SonarQube los marcaría para revisión manual):
  - El archivo `.env` está **versionado en Git** y contiene `JWT_SECRET=secret` (secreto trivial) y credenciales de ejemplo. Recomendación: agregar `.env` a `.gitignore` y publicar solo un `.env.example`.
  - `app.enableCors()` en `main.ts` permite cualquier origen. Recomendación: restringir orígenes.

### 1.3 Calidad / fiabilidad
- El código compila y las 13 pruebas pasan.
- La cobertura es baja (39 %) porque en el último commit se eliminaron los tests de `Espacio` y `Reserva`: `espacio.*`, `reserva.*`, `auth`, `config` y `main.ts` tienen 0 % de cobertura; solo el módulo `solicitud` llega al 100 %.

### 1.4 Plan de acción priorizado
1. Actualizar dependencias vulnerables (impacto alto, esfuerzo medio).
2. Quitar `.env` del repositorio y rotar el secreto JWT (impacto alto, esfuerzo bajo).
3. Restaurar pruebas de `Espacio` y `Reserva` para superar 80 % de cobertura.
4. Extraer *fixtures* de test compartidos y eliminar imports sin usar (reduce duplicación y code smells).

---

## 2. Proyecto propio (ejemplo): `proyecto-propio`

Gestor de tareas en TypeScript (54 líneas, 6 pruebas). Es un proyecto de **ejemplo** incluido para demostrar el flujo; se puede reemplazar por el proyecto real del estudiante.

| Condición de calidad | Valor medido | Estado |
|---|---|---|
| Compila sin errores | sin errores | APROBADO |
| Pruebas unitarias | 6 pasan de 6 | APROBADO |
| Cobertura ≥ 80 % | 100 % | APROBADO |
| Duplicación ≤ 3 % | 0 % | APROBADO |
| Vulnerabilidades críticas/altas | 0 | APROBADO |
| Problemas de seguridad en código | 0 | APROBADO |
| Code smells | 0 | APROBADO |

**Resultado: puerta de calidad APROBADA.**

---

## 3. Conclusiones

- El análisis estático automático permitió, en minutos, cuantificar la mantenibilidad, seguridad y calidad de un proyecto ajeno y obtener un plan de mejora concreto.
- Un proyecto con pruebas que pasan puede, aun así, no cumplir la puerta de calidad (caso `nestjs-espacios`): la cobertura, la duplicación y las dependencias son riesgos para la continuidad y el mantenimiento del software.
- Integrar este análisis en CI (`.github/workflows/analisis-estatico.yml`) hace que cada cambio sea evaluado automáticamente, que es el objetivo de la gestión del cambio.

Resultados crudos: carpeta `reports/` (JSON de ESLint, `npm audit`, cobertura y duplicación).
