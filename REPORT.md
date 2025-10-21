# Lab 3 — Complete a Web API — Project Report

## Description of Changes

He completado los **TODOs** de `ControllerTests.kt` para ejercitar el `EmployeeController` y **demostrar seguridad e idempotencia** de los verbos HTTP mediante *stubs* y *verifications* con **MockK** + **MockMvc** (sin tocar el controlador). El lab pide terminar los bloques **SETUP/VERIFY** y ofrece *provided code blocks* como guía; los he seguido y adaptado a nuestros casos de POST/GET/PUT/DELETE. 

**Cambios clave por test**  

- **POST is not safe and not idempotent**: Stub de `repository.save(...)` devolviendo IDs distintos con `answers { id=1 } andThenAnswer { id=2 }`. Se validan **201 Created**, `Location` diferentes (`/employees/1` y `/employees/2`) y `verify(exactly = 2) { save(any()) }` para reflejar que **no es idempotente** (cada POST crea un recurso nuevo). fileciteturn2file0 fileciteturn2file2  
- **GET is safe and idempotent**: Stub de `findById(1)` → `Optional.of(Mary)` y `findById(2)` → `Optional.empty()`. Se hacen dos GET al id=1 y uno al id=2. Se valida el **mismo JSON** en las dos lecturas y que **no hay writes**: `verify(exactly = 0) { save(..); deleteById(..) }`. **Seguro** e **idempotente**. fileciteturn2file0  
- **PUT is idempotent but not safe**: Encadenado de `findById(1)` → `Optional.empty()` **y luego** `Optional.of(Tom)`, más `save(..)` → `Employee(Tom, id=1)`. 1ª vez **201 Created** (crea), 2ª vez **200 OK** (repite misma representación); ambas devuelven el **mismo JSON** y el mismo `Content-Location`. `verify(exactly = 2) { findById(1) }` y `verify(exactly = 2) { save(any()) }`. **Idempotente** (repetir deja el mismo estado) pero **no seguro** (hay writes). fileciteturn2file0  
- **DELETE is idempotent but not safe**: `justRun { deleteById(1) }` para método *void*. Dos DELETE devuelven **204 No Content** y `verify(exactly = 2) { deleteById(1) }` + cero invocaciones a otros métodos. **Idempotente** (repetir no cambia más el estado) pero **no seguro** (borrado). fileciteturn2file0

> Notas de tooling: el manual del lab recuerda que **Ktlint** puede re-formatear y romper el build si no se lanza antes; por eso incluyo `./gradlew ktlintFormat` en los pasos de ejecución. fileciteturn2file4

---

## Technical Decisions

- **Estrategia de pruebas (MockMvc + MockK)**  
  - **Seguridad** se comprueba garantizando **ausencia de efectos**: `verify(exactly = 0)` sobre métodos de escritura (`save`, `deleteById`, `findAll`), tal como aconsejan los bloques de verificación del enunciado.
  - **Idempotencia** se evidencia repitiendo **la misma petición** y comprobando **misma representación** (cuerpo JSON y URI canónica) aun cuando el **status code** pueda cambiar de **201→200** en PUT, lo cual es correcto semánticamente. Para POST, se comprueba **no idempotencia** con `Location` distinto (ids distintos).  
  - **Stubs encadenados** con `andThenAnswer` para simular **cambios de estado entre llamadas**: en POST (IDs 1→2) y en PUT (`findById` vacío→presente). En DELETE, `justRun { deleteById(..) }` por ser `Unit`. Los patrones vienen directamente en “Provided code blocks”.
- **Disciplina de formato**  
  - Se ejecuta `ktlintFormat` antes del build para no fallar por re-formateo automático, como indica el lab. fileciteturn2file4

---

## Learning Outcomes

- **Diferencia entre “seguro” e “idempotente” aplicada a código**  
  - **Seguro**: no hay *writes* (p.ej., GET).  
  - **Idempotente**: repetir deja el mismo estado observable (GET/PUT/DELETE). **POST no lo es** porque crea nuevas entidades cada vez. El propio handout resume estas definiciones. fileciteturn2file4
- **Por qué en PUT hay 2 `save` en el test**  
  - En **un mismo test** se ejecutan **dos llamadas PUT**: la 1ª **crea** (no existía) y hace `save`; la 2ª **actualiza/no‑op** y también hace `save` por implementación, aunque la representación final no cambie. Ser idempotente **no significa** “no escribir la segunda vez”, sino que **repetir no cambia el resultado final** (misma entidad/JSON). Esta distinción me ayudó a consolidar el concepto.  
- **Patrones de MockK que me llevo**  
  - `every { ... } answers { ... } andThenAnswer { ... }` para simular evolución del estado.  
  - `justRun { ... }` para métodos `Unit`.  
  - `verify(exactly = n)` para cuantificar llamadas y justificar seguridad/idempotencia siguiendo el guion del lab. fileciteturn2file0  
- **Testing de cabeceras HTTP**  
  - Validar `Location`/`Content-Location` y los códigos **201/200/204/404** me ha dado una visión más precisa de **contratos REST** y de cómo documentarlos/pruebas de regresión.

> Tabla resumen final (según HTTP y lo evidenciado por los tests):  
> 
> | Método | Seguro | Idempotente | Evidencia principal                             |
> | ------ | ------ | ----------- | ----------------------------------------------- |
> | GET    | ✅      | ✅           | Misma respuesta dos veces, **0 writes**         |
> | POST   | ❌      | ❌           | `Location` distinto (IDs 1→2), `save`×2         |
> | PUT    | ❌      | ✅           | 201→200 con **mismo JSON** y `Content-Location` |
> | DELETE | ❌      | ✅           | 204 dos veces, `deleteById`×2                   |

---

## How to run & verify

```bash
# Formateo recomendado por el lab (evita fallos por ktlint)
./gradlew ktlintFormat

# Ejecutar tests
./gradlew test
```

Criterios del lab para “How to Pass”: **branch principal con tareas completas**, test **en verde** (incluida CI). fileciteturn2file2

---

## AI Disclosure

De acuerdo con los requisitos del propio handout, incluyo una declaración clara. 

### AI Tools Used

- **ChatGPT (GPT‑5 Thinking)**

### AI‑Assisted Work

- Explicaciones y *debugging* conceptual sobre **seguridad vs idempotencia** (incluida la duda de por qué hay dos `save` en PUT).  
- Redacción de este **REPORT.md** (edición final por mí).

**Estimación**: ~**35% asistido** / **65% original** (integración con mi repo, ajustes, ejecución de pruebas, verificación de cabeceras y estados).

### Original Work

- Implementación concreta de los stubs/verificaciones en mi `ControllerTests.kt`, elección de *setups* por verbo, y validación de resultados **201/200/204/404** y cabeceras (`Location`/`Content-Location`).  
- Revisión manual del controlador y contraste con las definiciones HTTP del handout.  
- Ejecución de `ktlintFormat` + `test` y documentación de decisiones/aprendizajes.
