# Respuestas · Laboratorio 4
Alumno: [20171857]
Presupuesto: [ 8 ] min  Métrica DORA: [ change failure rate ]

## Parte 1 · Diagnóstico
### Defecto 1
- Job y etapa: smoke, deploy.
- Problema que causa en el pipeline: smoke (y también e2e y load) no esperan a que deploy termine, por lo que pueden probar una versión no actualizada. Esto hace que se deje pasar un run verde que certificó un artefacto distinto al deploy que se acaba de publicar. 
- Link al commit: https://github.com/Setziret/INF384-lab4-20171857/commit/ca4ac24904b7a72a2950194dbfa73c90e9d24b61
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Ajustar la configuración cambiando need build-push a need deploy. 
- Evidencia (dato observado, con link al run o fragmento de la salida local):
https://github.com/Setziret/INF384-lab4-20171857/actions/runs/40956362754
En la gráfica de actions smoke y deploy son dos cajas separadas. 
En ci.yml:
deploy:
  needs: build-push
smoke:
  needs: build-push

### Defecto 2
- Job y etapa: smoke, etapa post-deploy. 
- Problema que causa en el pipeline: En test/smoke/smoke.sh se usa una variable llamada EXPECTED_COMMIT, que nunca aparece o se declara en smoke en ci.yml. Viendo como funciona el condicional, este bloque nunca se ejecuta por lo que la prueba de humo, aunque entrega un commit, este no necesariamente es el de la versión publicada, por lo que daria una falsa confianza. 
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Agregar EXPECTED_COMMIT: ${{github.sha}} a smoke en ci.yml. 
- Evidencia (dato observado, con link al run o fragmento de la salida local):

En #test/smoke/smoke.sh se ve: 
    if [ -n "${EXPECTED_COMMIT:-}" ]; then

No se llega a comparar el commit aunque se entregue uno, como se ve en el run. 
salud: {"ok":true,"version":"1.0.0","commit":"49ba49ebee84f4c9f8b6943031595970311a3bb6"}


### Defecto 3
- Job y etapa: load, etapa post-deploy.
- Problema que causa en el pipeline: En test/load/transferir.js se define un perfil de stages de 10 min por 30 usuarios virtuales y el load corre todos los eventos sin condiciones. Esto hace que se pierda tiempo y es lo que se ve cuando se corre el workflow, pues en load es donde más se demora. Esto impide pushear cambios continuos. 
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Hacer que no corra en cada pull request, pero si en push a main o en workflow_dispatch. 
- Evidencia (dato observado, con link al run o fragmento de la salida local):
Este es el primer run sin corregir el defecto, donde se ve que load demora como 12 minutos o más. 
https://github.com/Setziret/INF384-lab4-20171857/actions/runs/40956362754

test/load/transferir.js
export const options = {
  stages: [
    { duration: '1m', target: 30 },
    { duration: '10m', target: 30 },
    { duration: '1m', target: 0 },

### Defecto 4
- Job y etapa: sca, etapa pre-build
- Problema que causa en el pipeline: El paso "Trivy" usa --include-dev-deps, por lo que también se evalúa @playwright/test, c9 y esbuild. En el Dockerfile se hace un npm prune --omit=ev antes de armar la imagen final por lo que ninguna dependencia se va al artefacto publicado. Esto causa que si se encuentra un hallazgo HIGH o CRITICAL puede bloquear la publicación de un artefacto que no contiene las dependencias, por lo que no es un riesgo real y hace perder tiempo tratando de diagnosticar o entender que pasó. 
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests): Borrar la linea de include-dev-deps de sca.
- Evidencia (dato observado, con link al run o fragmento de la salida local):
En /.github/workflows/ci.yml
    - name: Trivy
        run: |
          docker run --rm \
            --volume "$PWD:/src" \
            --workdir /src \
            ghcr.io/aquasecurity/trivy:0.76.1 fs \
              --scanners vuln \
              --include-dev-deps \
              --severity HIGH,CRITICAL \
              --exit-code 1 \
              package-lock.json
En Dockerfile:
    RUN npm run build && npm prune --omit=dev

### Defecto 5
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):

## Parte 2 · Implementación
- Link al run en verde en main:
- Link al pull request:
- Defecto de la aplicación introducido (archivo y descripción):
- Job que debería detenerlo:
- Link al run fallido:

## Parte 3 · Decisión bajo restricción
Link al run medido:

| Job | Duración (min:seg) |
|---|---|
| unit | |
| integration | |
| sast | |
| sca | |
| build-push | |
| deploy | |
| smoke | |
| e2e | |
| load | |
| **Total** | |

Resultado de load en ese run:
| Métrica | Valor |
|---|---|
| p90 | |
| p95 | |
| Máximo | |
| Tasa de error | |
| Requests por segundo | |

Conclusión sobre el comportamiento bajo carga:

SLO propuesto (percentil, valor, ventana) y justificación:

SLA propuesto y diferencia con el SLO:

Cambios para cumplir el presupuesto:
| Cambio | Job | Minutos recuperados |
|---|---|---|

Total resultante: [ ] min
Link al run en main con los cambios:

Riesgo aceptado (defecto de la aplicación concreto y efecto en la métrica DORA asignada):

Decisión sobre sast (se queda / se mueve a otra etapa o evento / informativo) y razón:

## Declaración de uso de IA
Herramientas usadas y para qué:
Se usó Claude chat antes del laboratorio para examinar todo el repositorio y, viendo la plantilla de RESPUESTAS.md, examinar los posibles defectos que habrían puesto que los laboratorios suelen manejarse así. La herramienta me dio posibles preguntas para el laboratorio y consejos acerca de qué hacer antes de iniciar. En particular, me ayudó a identificar el defecto 4, pues involucraba bastantes cosas. 

