# Refutadores en frío: las plantillas y lo que medimos

*Cold refuters: the prompts and what we measured. English summary at the end.*

Esto amplía el patrón 3 de la guía, la [verificación adversaria](README.md#3--verificación-adversaria--adversarial-verification). Antes de dar por bueno un cambio importante (código, una automatización, una conclusión sacada de datos), **dos agentes con el contexto limpio intentan demostrar que está mal**. Cada uno mira desde un ángulo distinto.

## Las dos plantillas

Se lanzan a la vez, cada una como un subagente nuevo. Ninguno de los dos sabe quién hizo el trabajo, qué se esperaba ni qué quiere oír quien lo pide. Solo recibe el material.

### Refutador A: «¿el fallo puede seguir pasando?»

```text
Hoy es <fecha>. Eres un revisor escéptico y no has visto este trabajo antes.

Contexto: <qué fallaba, en dos líneas, con el síntoma real>
Material: <el cambio, el diff, el script o la conclusión, y las rutas o datos para comprobarlo>

Tu encargo: demostrar que el fallo PUEDE SEGUIR PASANDO tal como está.
Busca el caso que el arreglo no cubre: otra entrada, otra ruta, otro momento, otra instancia,
una variante (mayúsculas, tildes, plurales, zona horaria, fichero vacío, servicio caído).

Reglas:
- Cada hallazgo, con la comprobación que lo sostiene (qué has abierto o ejecutado y qué has visto).
- Marca como BLOQUEANTE lo que haría que el arreglo no sirva.
- Si dudas, decide que está mal.
- No propongas mejoras de estilo. Solo lo que haría que el fallo vuelva.
```

### Refutador B: «¿esto rompe otra cosa?»

```text
Hoy es <fecha>. Eres un revisor escéptico y no has visto este trabajo antes.

Contexto: <qué se ha cambiado y para qué, en dos líneas>
Material: <el cambio y lo que lo rodea: quién lo llama, qué lee, qué escribe, qué programa lo usa>

Tu encargo: demostrar que este cambio ROMPE OTRA COSA.
Mira lo que depende de él, lo que sobrescribe, lo que deja de avisar, cómo se deshace si sale
mal y qué pasa si se ejecuta dos veces.

Reglas:
- Cada hallazgo, con la comprobación que lo sostiene.
- Marca como BLOQUEANTE lo que causaría daño en producción o no se podría deshacer.
- Si dudas, decide que está mal.
- No repitas el trabajo original. Busca los efectos secundarios.
```

**Qué se hace con lo que devuelven:** lo que se demuestra se corrige y lo que no se sostiene se descarta, diciendo por qué. Si tumban el mismo arreglo dos veces seguidas, hay que parar y replantearlo, no darle otra vuelta más.

## ¿Hacen falta dos agentes o basta con uno?

Un solo agente con los dos encargos sale a mitad de precio, y lo que se ahorra no es poco: en la semana que medimos, los refutadores eran **algo más de una cuarta parte** (27 %) del gasto total en Claude Code. Merecía la pena medirlo en vez de suponerlo.

### Fase 1: lo que ya había pasado

Revisamos 94 situaciones reales en las que se habían lanzado dos refutadores. En **78 (un 83 %)** cada uno encontró algo real que el otro no vio.

Pero en todas, los dos tenían **encargos distintos**. Eso demostraba que **dos encargos** rinden más que uno. No demostraba que hicieran falta **dos agentes**: nunca se había probado uno solo con los dos encargos.

### Fase 2: la prueba de verdad

En los 8 trabajos reales siguientes se lanzaron **tres** refutadores a la vez: A, B y un tercero, C «combinado», que recibía los dos encargos en un mismo prompt. Los tres con el mismo modelo, el mismo material y sin saber que había otros.

**La regla de decisión se escribió antes de ver ningún dato**, para no hacernos trampas:

- Si C encuentra todos los bloqueantes **y** al menos el 90 % de lo que encuentran A y B juntos, en 7 de los 8 trabajos o más, se pasa a un solo agente en lo de bajo riesgo.
- Si falla en 2 trabajos o más, se quedan los dos.

Los trabajos eran variados: cambios en campañas de anuncios, filtros de contenido, vigías de sistemas, manuales, mantenimiento de servidores y la memoria del propio agente.

| | Resultado |
|---|---|
| Bloqueantes encontrados por C | **Todos**, en los 8 trabajos |
| Del total que encontraron A y B juntos | **82 de 114 (72 %)** |
| Peor trabajo / mejor trabajo | 50 % / 89 % |
| Trabajos en los que C llegó al 90 % | **Ninguno** |

**Decisión: se quedan los dos agentes.** El combinado no se deja nada grave, pero se le escapan cerca de 3 de cada 10 problemas «menores». Y esos son justo los que, en producción, acaban siendo la avería de dentro de dos semanas.

La explicación más probable es que, con los dos encargos juntos, el agente acaba tirando hacia uno y el otro se queda a medias. Con un encargo cada uno, se mira de verdad en dos direcciones.

**Lo que sí cambiamos:** los refutadores van con el modelo estándar, no con el más caro. Ahorrar en el número de agentes salía caro. Ahorrar en el modelo, no.

## Límites de la medición

- 8 trabajos es poco. Da para decidir en un sentido, no para dar una cifra exacta.
- «Hallazgo» es un problema real que se corrigió o se aceptó con prueba. Lo que se descartó no cuenta.
- Es una sola forma de trabajar, con un solo tipo de sistemas. Si lo mides en la tuya, me interesa el resultado.

---

## English summary

Two refuter agents with a clean context try to prove a change wrong before it counts as right. One asks *"can the bug still happen?"*, the other *"does this break something else?"*. The prompts are above.

We tested whether one agent holding both briefs could replace the two. Over 8 real jobs, with the decision rule written in advance, the combined agent caught **every blocking issue** but only **82 of 114 (72%)** of what the two separate agents found together, and never reached our 90% bar in any job. **We kept two agents**, and saved money by using the standard model instead of the premium one.

---

Si esto te sirve, una ⭐ al repositorio ayuda a que le llegue a más gente que trabaja con agentes en español.
