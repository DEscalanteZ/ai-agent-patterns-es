# Seis patrones para trabajar con agentes de IA (y el bucle que los sostiene)

> 🇬🇧 **[Full English version → README.en.md](README.en.md)**

> **In English, in one paragraph:** a field guide, in Spanish, to the six ways of organizing work when you have several AI agents instead of one: classify-and-act, fan-out-and-synthesize, adversarial verification, generate-and-filter, tournament, and loop-until-done. Plus the gather → act → verify loop underneath them. Each pattern comes with a plain-language explanation, a real use case, **its trap**, and what it costs. Written from running dozens of production automations across three companies, where almost every serious outage was a job that "succeeded" and produced nothing.

Guía práctica para quien dirige agentes de IA sin ser necesariamente programador. La taxonomía de los seis patrones viene de las infografías de la entrevista a **Boris Cherny**, el creador de Claude Code. Las explicaciones, los ejemplos y las trampas son nuestros, sacados de meses con automatizaciones en producción.

---

## Glosario mínimo

| Término | Qué es |
|---|---|
| **Agente** | Una IA que trabaja sola en una tarea, con sus propias herramientas |
| **Subagente** | Un agente que otro agente lanza para una parte del trabajo y que le devuelve el resultado |
| **Abanico** (*fan-out*) | Repartir un trabajo entre muchos agentes a la vez |
| **Rúbrica** | La lista de criterios con la que se puntúa algo |

---

## Primero: el bucle

```
   Reunir contexto  →  Actuar  →  Comprobar el trabajo
          ↑___________________________________|
```

1. **Reunir contexto**: mirar los ficheros, leer los registros, ver qué hay de verdad.
2. **Actuar**: escribir el código, mandar el aviso, mover el fichero.
3. **Comprobar el trabajo**: verificar que salió lo que tenía que salir. Si no, se vuelve a empezar.

### Por qué es lo más importante de todo

**Casi todas las averías serias que hemos tenido fueron el paso 3 saltado.** Los robots no fallaron: corrieron, terminaron en verde, y nadie miró qué habían producido.

- Una copia de seguridad que corría todos los días y llevaba una semana sin copiar nada.
- Un proceso contable que terminaba «bien» con el fichero de salida vacío.
- Tareas programadas duplicadas que entraban dos veces al mismo sitio, y cada una «funcionaba».
- Un vigía que desapareció de la lista del guardián, mientras la salud global seguía en verde: el guardián contaba lo que veía, no lo que faltaba.

La regla que sale de ahí: **comprobar qué produjo, nunca solo que corrió.** Un «OK», un código de salida 0 o un fichero con fecha de hoy no demuestran nada, porque pueden venir vacíos.

---

## Los seis patrones

### 1 · Clasificar y repartir · *Classify-and-Act*

**Qué es:** llega algo, un clasificador decide de qué tipo es y lo manda al especialista que toca. Funciona como una centralita.

**Ejemplo:** llegan correos a varios buzones de varias empresas. Un clasificador decide de qué empresa es cada uno y lo marca con un semáforo. Nadie lee trescientos correos: el clasificador reparte.

**La trampa:** si el clasificador falla, todo lo que va detrás falla en silencio y convencido. Al clasificador se le mide **por separado**, con un puñado de casos que ya sabes clasificar a mano.

**Cuánto cuesta:** poco. El clasificador puede ser el modelo más barato que aguante.

### 2 · Abrir en abanico y juntar · *Fan-out-and-Synthesize*

**Qué es:** una tarea grande se parte en trozos, cada trozo va a un agente distinto **a la vez**, y al final uno lo junta todo en una sola respuesta.

**Ejemplo:** un parte semanal que revisa una docena de vigías. En vez de recorrerlos en fila, cada bloque se mira en paralelo y el último agente redacta el parte.

**La trampa:** es un enjambre. Ocho agentes cuestan ocho veces más. Solo sirve cuando los trozos **son de verdad independientes**: si el trozo 2 necesita lo que encontró el trozo 1, no vale.

**Cuánto cuesta:** es el más caro de los seis. Merece una regla explícita: nada de enjambres grandes sin el visto bueno de quien paga.

### 3 · Verificación adversaria · *Adversarial Verification*

**Qué es:** uno hace el trabajo. Otros, que no saben quién lo hizo ni qué se esperaba, intentan **demostrar que está mal**. Si aguanta, se da por bueno.

**Ejemplo, y en nuestra casa es regla:** el *revisor en frío*. Ningún hallazgo de peso (un fallo, una cifra en euros, una conclusión de negocio) se da por bueno hasta que otro agente, con el contexto limpio y el encargo de refutar, ha intentado tumbarlo. Si duda, decide que es falso.

**Por qué existe:** si le pides a una IA que encuentre un fallo, encuentra uno **aunque no exista**, porque tiende a complacer. El antídoto barato es que otro intente destrozarlo.

**El matiz:** cuando algo puede fallar de varias maneras, rinden más **revisores con ángulos distintos** (¿es correcto?, ¿se reproduce?, ¿rompe otra cosa?) que varios iguales. Varios iguales se equivocan igual.

**Las plantillas y lo que medimos:** los dos prompts que usamos y una prueba con 8 trabajos reales para ver si un solo agente con los dos encargos bastaba. No bastó. Está en [refutadores-en-frio.md](refutadores-en-frio.md).

### 4 · Generar mucho y filtrar · *Generate-and-Filter*

**Qué es:** varios agentes producen muchos candidatos, y después un filtro con criterios escritos se queda con los buenos y descarta el resto.

**Ejemplo:** la criba de una revisión sistemática. Entran miles de artículos y un filtro con criterios PRISMA decide cuáles pasan.

**La trampa:** **el filtro vale lo que valen sus criterios.** Si la rúbrica sale de la cabeza en vez de leerse de la fuente, la criba se queda ciega sin que se note: sigue saliendo algo, solo que lo equivocado. Los criterios se escriben **antes** de mirar los candidatos, y se prueban con casos que ya sabes clasificar.

### 5 · Torneo · *Tournament*

**Qué es:** se hacen varios intentos completos del mismo trabajo, se comparan **de dos en dos** y el ganador de cada pareja sube. Al final queda uno.

**Ejemplo:** para una pieza de diseño que importa, se preparan varias variantes con estilos opuestos y se eligen por parejas.

**Por qué de dos en dos:** comparar dos cosas es mucho más fiable que ponerle un 7,5 a una. Sabes cuál de dos portadas te gusta más antes de saber por qué.

**La trampa:** el juez hereda los gustos de quien escribió la rúbrica. Si el criterio dice «moderno», gana lo moderno, no lo que vende. En lo comercial, el juez debería ser un dato de ventas, no una opinión.

### 6 · Dar vueltas hasta que no salga nada nuevo · *Loop Until Done*

**Qué es:** el agente busca, y si encuentra algo nuevo, lanza otra ronda. Cuando dos rondas seguidas no encuentran nada, para. **No para por un número fijo de intentos, sino porque ya no queda nada que encontrar.**

**Por qué importa:** si pides «búscame 10 fallos», te encuentra 10 y se calla, aunque hubiera 30 o solo 4 de verdad. El número fijo engaña por los dos lados.

**Ejemplo:** una auditoría que dio el recuento por bueno tras mirar un almacén de datos, cuando había dos. Una segunda ronda con la pregunta «¿qué me falta por mirar?» lo habría detectado.

**La trampa:** hay que apuntar todo lo **encontrado**, no solo lo aprobado. Si solo recuerdas lo que pasó el filtro, lo rechazado vuelve a salir en cada ronda y no se acaba nunca. Y hay que fijar un **techo de rondas** antes de empezar.

---

## Lo que te llevas

1. **Seguramente ya usas varios** con otros nombres. No hace falta empezar de cero: basta con ponerles nombre y usarlos a propósito.
2. **El que más ahorra es el bucle completo**, sobre todo su tercer paso. Casi todas las averías son el mismo agujero: nadie comprobó **qué salió**.
3. **El que hay que usar con cuidado es el abanico**, porque multiplica el gasto.
4. **Una advertencia sobre los patrones:** son formas de organizar el trabajo, no inteligencia. Un torneo entre cuatro respuestas malas da la menos mala. Ninguno arregla unos datos de entrada malos. Eso solo se arregla mirando la fuente.

---

## Más trabajos del mismo autor

- **[Bitácora de averías](https://github.com/DEscalanteZ/bitacora-de-averias-es)**: las tres formas en que fallan las automatizaciones con IA (130 averías reales) y la regla del candado.
- **[Dos Claude Code que se pasan trabajo por ficheros](https://github.com/DEscalanteZ/claude-code-buzon-es)**: un canal entre dos agentes con buzones en Markdown, hooks y pruebas.

---

⭐ **Si esta guía te sirve, dale una estrella al repositorio**: es la forma más sencilla de que le llegue a más gente que trabaja con agentes en español.

*Autor: David Escalante ([@DEscalanteZ](https://github.com/DEscalanteZ)). La taxonomía de los seis patrones viene de la entrevista a Boris Cherny (Anthropic). Licencia: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.es): puedes copiarla y adaptarla citando la fuente.*
