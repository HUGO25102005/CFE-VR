# CFE-VR — Simulador de Entrenamiento en Realidad Virtual

> Proyecto de entrenamiento inmersivo en VR para operadores de CFE.  
> Desarrollado con **Unity + XR Interaction Toolkit**.

---

## Tabla de Contenidos

1. [Arquitectura y Reglas del Proyecto](#1-arquitectura-y-reglas-del-proyecto)
2. [Estructura de Directorios](#2-estructura-de-directorios)
3. [Flujo de Trabajo — Integrar Maquinaria Nueva](#3-flujo-de-trabajo--integrar-maquinaria-nueva)
4. [Convenciones de Nomenclatura](#4-convenciones-de-nomenclatura)
5. [Optimización VR](#5-optimización-vr)

---

## 1. Arquitectura y Reglas del Proyecto

La arquitectura adoptada es **Modular Interaction-Based (SO-Driven)**.  
El objetivo es mantener el proyecto escalable, libre de dependencias rígidas y fácil de extender sin romper lo ya construido.

Todo el desarrollo propio **debe vivir dentro de `Assets/_Project`**. Nunca se deben modificar carpetas de Plugins externos.

---

### Regla 1 — Desacoplamiento mediante ScriptableObjects

| ❌ Prohibido | ✅ Obligatorio |
|---|---|
| `GameObject.Find()` para comunicar objetos | Usar **Game Events** basados en ScriptableObjects |
| Referencias directas pesadas en el Inspector para comunicación entre sistemas | El sistema emisor dispara un `SO Event`; el receptor lo escucha y reacciona |

**Por qué importa:** permite cambiar máquinas o pasos del tutorial sin modificar ni romper el código existente.

---

### Regla 2 — Lógica de Interacción Atómica

Cada script debe realizar **una sola tarea**.

```
❌  MachineManager.cs          (1000 líneas, hace todo)

✅  MachineAnimationController.cs   → Solo visual / animaciones
    MachineAudioHandler.cs          → Solo sonido
    MachineInteractionTrigger.cs    → Solo lógica de colisión / agarre
```

---

### Regla 3 — El "Estado" del Tutorial es Externo

La progresión del tutorial (Paso 1, Paso 2…) **no debe vivir en scripts pegados a objetos 3D**.  
Debe residir en **ScriptableObjects de Datos** (`SO_Step`).

**Beneficio:** crear un nuevo tutorial equivale a crear nuevos archivos de datos, sin programar de nuevo.

---

### Regla 4 — UI Exclusiva en World Space

En VR **no existe el Canvas Overlay**. Toda interfaz debe estar en espacio 3D (**World Space**).

> **Regla de oro:** la UI debe ser parte del entorno (pantallas de la máquina, tablets flotantes) para mantener la inmersión del operador.

---

### Regla 5 — Uso Estricto de Prefabs y Variantes

| ❌ Prohibido | ✅ Obligatorio |
|---|---|
| Modificar objetos directamente en la escena | Todo objeto interactuable debe ser un **Prefab** |
| Duplicar Prefabs y editarlos por separado | Usar **Prefab Variants** para versiones alternativas |

**Beneficio:** arreglar un bug en el Prefab base lo corrige automáticamente en todas las escenas.

---

### Regla 6 — Nomenclatura Estándar

Ver sección [4. Convenciones de Nomenclatura](#4-convenciones-de-nomenclatura).

---

### Regla 7 — Capas de Interacción (Interaction Layers)

Configurar correctamente las **Interaction Layers del XR Interaction Toolkit**.

- Un objeto **no disponible en el Paso 1** debe tener su capa desactivada por código.
- La capa se activa cuando el tutorial avanza al paso correspondiente.

**Beneficio:** impide que el usuario se salte instrucciones o interactúe con objetos fuera de contexto.

---

### Regla 8 — Optimización de Assets (VR Ready)

Ver sección [5. Optimización VR](#5-optimización-vr).

---

## 2. Estructura de Directorios

```
Assets/
├── _Project/
│   ├── Art/
│   │   ├── Environment/        # Escenarios estáticos
│   │   ├── Machines/           # Maquinaria pesada e interactiva
│   │   └── Props/              # Herramientas y objetos pequeños
│   ├── Audio/                  # SFX y Voces de guía (Voice Over)
│   ├── Prefabs/
│   │   ├── Core/               # Player VR, Managers, Sistemas globales
│   │   └── Interactables/      # Objetos con los que el usuario interactúa
│   ├── Scenes/                 # Niveles o módulos de entrenamiento
│   ├── ScriptableObjects/      # El "cerebro" de la arquitectura (Eventos/Datos)
│   └── Scripts/
│       ├── Core/               # Sistemas globales (Architecture logic)
│       ├── Gameplay/           # Lógica específica del tutorial / maquinaria
│       ├── Interactions/       # Scripts de comportamiento VR
│       └── UI/                 # Interfaces en World Space
└── Plugins/                    # XR Interaction Toolkit, Oculus SDK, etc. (NO MODIFICAR)
```

---

## 3. Flujo de Trabajo — Integrar Maquinaria Nueva

Sigue estos pasos en orden cada vez que se agrega una máquina al proyecto:

1. **Importación**  
   Colocar el modelo 3D en `Assets/_Project/Art/Machines/`.

2. **Prefabricación**  
   - Crear el Prefab en `Assets/_Project/Prefabs/Interactables/`.  
   - Añadir el componente correspondiente: `XR Grab Interactable` o `XR Simple Interactable`.  
   - Nombrar siguiendo la convención: `PFB_NombreMaquina`.

3. **Eventos**  
   - Crear los `ScriptableObjects` de tipo evento necesarios en `Assets/_Project/ScriptableObjects/`.  
   - Estos SOs permiten que la máquina se comunique con el `TrainingManager` sin dependencias directas.  
   - Nombrar como: `SO_NombreEvento`.

4. **Instrucción de Tutorial**  
   - Crear un nuevo `SO_Step` (Paso de Tutorial) que referencie la acción de la máquina.  
   - Nombrar como: `DATA_StepN_NombreAccion`.

---

## 4. Convenciones de Nomenclatura

| Elemento | Convención | Ejemplo |
|---|---|---|
| Scripts C# | `PascalCase` | `HandController.cs` |
| Variables privadas | `camelCase` con prefijo `_` | `_isOperating` |
| Prefabs | `PFB_NombreObjeto` | `PFB_TransformadorMT` |
| ScriptableObjects — Eventos | `SO_NombreEvento` | `SO_MachineActivated` |
| ScriptableObjects — Datos | `DATA_Configuracion` | `DATA_Step01_EncenderMaquina` |
| Scenes | `SCN_NombreModulo` | `SCN_Modulo01_Introduccion` |

---

## 5. Optimización VR

El rendimiento es crítico en VR para evitar mareos (motion sickness). Aplicar siempre:

| Área | Lineamiento |
|---|---|
| **Polígonos** | No exceder la carga poligonal necesaria para el nivel de detalle requerido |
| **Texturas** | Usar el formato comprimido adecuado para la plataforma objetivo (Oculus / PCVR) |
| **Static Flag** | Marcar como `Static` todo objeto que no se mueva, para habilitar el baking de luces |
| **Draw Calls** | Agrupar materiales compartidos; usar GPU Instancing donde sea posible |
| **Shadows** | Limitar sombras en tiempo real; preferir sombras bakeadas para geometría estática |

---

*Documento generado a partir de la Especificación Técnica: Arquitectura Modular de Interacción (VR CFE).*