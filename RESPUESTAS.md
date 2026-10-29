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
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):
### Defecto 4
- Job y etapa:
- Problema que causa en el pipeline:
- Link al commit:
- Corrección (una o más: mover de etapa / dejar como informativo / eliminar / ajustar configuración / modificar tests):
- Evidencia (dato observado, con link al run o fragmento de la salida local):
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
