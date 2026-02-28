# Laboratorio 3: Planning con Qwen3-8B

## Descripción

En este laboratorio diseñamos e implementamos una solución para resolver problemas de planificación utilizando el modelo **Qwen3-8B**, cumpliendo estrictamente las condiciones del enunciado.  

Nuestro sistema procesa `Task.json` y genera un archivo JSON de salida que contiene, para cada escenario:

- `assembly_task_id`
- `complexity_level`
- `target_action_sequence`

---

## Modelo utilizado

- Modelo: **Qwen/Qwen3-8B**
- Sin fine-tuning
- Cuantización: 4-bit (NF4) con bitsandbytes
- Inferencia determinista:
  - `temperature = 0.0`
  - `do_sample = False`

Usamos generación greedy para asegurar que los resultados sean completamente reproducibles durante la auditoría.

---

## Arquitectura de Prompts

Utilizamos una estrategia de **Few-Shot Chain-of-Thought (CoT) implícito**.

El `scenario_context` ya contiene varios ejemplos resueltos con la estructura:

[STATEMENT]  
[GOAL]  
[PLAN]  

El modelo observa el patrón:

estado -> goal -> secuencia de acciones

y luego completa el último `[PLAN]` correspondiente al task a resolver. No añadimos razonamiento artificial adicional, sino que aprovechamos los ejemplos ya incluidos en el contexto para que el modelo aprenda el patrón de resolución.

---

## Dominios

El dataset contiene dos dominios:

### Blocks
- (engage_payload X)
- (release_payload X)
- (mount_node X Y)
- (unmount_node X Y)

### Logic
- (attack X)
- (succumb X)
- (feast X Y)
- (overcome X Y)

Detectamos automáticamente el dominio según el texto del escenario y luego convertimos la salida en lenguaje natural del modelo a la notación formal requerida.

---

## Cálculo de `complexity_level`

No usamos umbrales arbitrarios.  

Leímos `Examples.json` y calculamos los rangos reales de acciones por cada nivel de complejidad. Observamos que:

- Nivel 2 → 2 acciones  
- Nivel 4 → 4 acciones  
- Nivel 6 → 6 acciones  

Luego asignamos el `complexity_level` según la cantidad de acciones generadas, usando esos umbrales aprendidos de los datos.

---

## Parámetros de generación

- `do_sample = False`
- `temperature = 0.0`
- `repetition_penalty = 1.1`
- `max_new_tokens = 150`

Esto realiza generación determinista, sin muestreo y evitando repeticiones innecesarias.

---

## Formato de salida

El archivo final (`output_planning.json`) tiene el siguiente formato:

[
  {
    "assembly_task_id": "...",
    "complexity_level": 6,
    "target_action_sequence": ["(...)"]
  }
]

Incluimos únicamente los campos solicitados en el enunciado.

---

## Cumplimiento de requisitos

- Usamos exclusivamente Qwen3-8B  
- No realizamos fine-tuning  
- Configuramos temperature = 0.0  
- Implementamos una arquitectura de prompts (Few-Shot CoT)  
- Calculamos `complexity_level` a partir de Examples.json  
- Generamos el formato JSON correcto  
- Diseñamos la solución para ejecutarse dentro del tiempo límite en Colab  

---

## Archivo principal

La implementación completa se encuentra en:

Laboratorio_3.ipynb
