# Propagación de una enfermedad respiratoria en la comunidad universitaria (Universidad de los Llanos)

Proyecto 8 · Simulación Computacional · Semestre 2026-II

Simulación de la propagación de una enfermedad respiratoria (tipo influenza o COVID-19) entre estudiantes, docentes y trabajadores, con movilidad entre aulas, salas de cómputo, comedor, biblioteca, oficinas y transporte. El modelo principal es **basado en agentes (ABM) en NetLogo o AnyLogic (aún por definir)**; como validación se incluye un modelo de **sistemas dinámicos (SD)** con estructura SEIR multigrupo.

- **Entrega y sustentación:** martes 17 de noviembre de 2026
- **Equipo:** Juan David Romero Villamizar (Modelador ABM)_, Astrid Daniela Martinez Ruiz (SD, datos y experimentos)_, Camilo Andrés Acosta Bernal (Documentación y presentación)_
- **Docente:** Ángel Cruz

---

## 1. Objetivo

Estimar, mediante simulación, el efecto de distintos escenarios sobre tres medidas de desempeño:

| Medida | Definición |
|---|---|
| **Pico** | Máximo de infectados simultáneos, `max I(t)` |
| **Duración del brote** | Días hasta que `E + I = 0` (ABM) o `E + I < 1` (SD); 112 si sigue activo al final del semestre |
| **Proporción infectada** | `(N − S_final) / N` |

## 2. Escenarios

| Escenario | Descripción |
|---|---|
| **A. Base** | Propagación libre, sin intervención, movilidad habitual durante un semestre regular (112 días) |
| **B. Variante** | Mayor transmisibilidad al inicio del semestre con alta densidad estudiantil |
| **C. Vacunación y aforos** | 70 % de la comunidad vacunada y aforo máximo en espacios cerrados (opcional) |

Las condiciones iniciales (población, infectados iniciales, semillas aleatorias) son iguales en todos los escenarios.

## 3. Estructura del repositorio

```
.
├── README.md
├── LICENSE
├── .gitignore
├── CHANGELOG.md
├── docs/
│   ├── guia/                  # guía paso a paso (.tex)
│   ├── odd/                   # descripción ODD del modelo
│   ├── articulo/              # informe en formato artículo (.tex, .bib, figuras)
│   └── presentacion/          # diapositivas y video de respaldo
├── modelo/
│   ├── abm_campus.nlogo       # ABM (modelo principal)
│   └── sd_campus.nlogo        # SD multigrupo (validación)
├── parametros/
│   ├── parametros.csv         # valor, rango, unidad y fuente de cada parámetro
│   └── referencias.bib        # bibliografía de los parámetros
├── experimentos/
│   ├── behaviorspace/         # configuraciones exportadas de BehaviorSpace
│   ├── calibracion/           # calibracion.csv (salidas crudas)
│   └── resultados/            # resultados.csv (salidas crudas por escenario)
├── analisis/
│   ├── calibrar_beta.py       # calibración de beta al R0 objetivo
│   ├── analizar_replicas.py   # medias, IC95 %, pruebas pareadas
│   ├── validar_seir.py        # SEIR de ecuaciones vs. ABM homogéneo
│   └── requirements.txt
└── figuras/                   # gráficas finales (PNG/PDF) usadas en el artículo
```

## 4. Requisitos

- **NetLogo 6.2 o superior** (usa `ifelse` con varias ramas y la extensión `table`)
- **Python 3.10+** con las librerías de `analisis/requirements.txt`:

```
pandas
numpy
scipy
matplotlib
```

- Una distribución de **LaTeX** u **Overleaf** para compilar la guía y el artículo.

## 5. Cómo ejecutar

### 5.1 ABM en NetLogo

1. Abrir `modelo/abm_campus.nlogo`.
2. Elegir el escenario en el selector `escenario` y ajustar los parámetros (ver `parametros/parametros.csv`).
3. Pulsar `setup` y luego `go`.
4. Para reproducir una corrida, fijar `semilla-exp`.

### 5.2 Calibración de `beta-base`

1. Activar `modo-calibracion?` y poner `inf-iniciales = 1`, escenario A y `p-aislar = 0`.
2. Ejecutar el experimento de BehaviorSpace `calibracion` (salida tipo *Table*) y guardar el resultado en `experimentos/calibracion/calibracion.csv`.
3. Ejecutar:

```bash
python analisis/calibrar_beta.py
```

4. Copiar el `beta-base` calibrado a `parametros/parametros.csv` y confirmar con una corrida de comprobación.

### 5.3 Experimento principal

1. Apagar `modo-calibracion?` y `modo-homogeneo?`.
2. Ejecutar el experimento `principal` (3 escenarios × 30 semillas emparejadas) y guardar en `experimentos/resultados/resultados.csv`.
3. Analizar:

```bash
python analisis/analizar_replicas.py
```

### 5.4 Validación frente al SEIR de ecuaciones

1. En el ABM: `modo-homogeneo? = true`, `k-gamma = 1`, `p-aislar = 0`, `inf-iniciales = 10`.
2. Exportar `I(t)` medio de 30 réplicas.
3. Ejecutar `python analisis/validar_seir.py` y comparar las curvas.

### 5.5 Modelo SD

Abrir `modelo/sd_campus.nlogo`, pulsar `setup-sd` y `go-sd`. Los parámetros y escenarios son los mismos del ABM (`beta-dia = 24 × beta-base`).

## 6. Parámetros

Todos los valores, con su rango, unidad y fuente, están en `parametros/parametros.csv`. Formato:

```
parametro,simbolo,valor,min,max,unidad,fuente
incubacion_media,T_inc,,,,dias,
periodo_infeccioso,T_inf,,,,dias,
R0,R0,,,,,
...
```

Regla del equipo: **ningún parámetro entra al modelo sin fuente** (o sin la nota «supuesto del equipo» y su justificación).

## 7. Supuestos principales

- Población simulada de _N_ agentes (muestra representativa) con 90 % estudiantes, 7 % docentes y 3 % trabajadores.
- Paso de tiempo de 1 hora; semestre de 112 días; fines de semana en casa.
- Solo los infectados (`I`) contagian; los expuestos (`E`) no.
- Aforos, factores de riesgo y agendas son supuestos del equipo (ver `docs/odd/`).
- Los parámetros provienen de la literatura; no se usan datos de campo.

## 8. Verificación y validación

- Pruebas de verificación: sin infectados, `beta = 0`, conservación `S + E + I + R = N`, un solo lugar, sin movilidad, efecto del aforo, vacunación perfecta.
- Calibración de `beta-base` al `R0` objetivo (500 réplicas por valor).
- Validación cruzada ABM homogéneo vs. SEIR de ecuaciones.

## 9. Flujo de trabajo del equipo

- Rama `main`: siempre estable. Cada integrante trabaja en su rama (`modelo-abm`, `modelo-sd`, `docs`) y abre *pull request*.
- **Congelamiento del modelo: 11 de noviembre de 2026.** Se crea la etiqueta `v1.0-modelo-congelado`; después solo se corrigen errores.
- Los experimentos finales se corren con el modelo etiquetado.
- Mensajes de commit breves y descriptivos, por ejemplo: `abm: agrega aforo maximo en espacios cerrados`.

## 10. Responsables

| Integrante | Rol | Carpetas principales |
|---|---|---|
| Juan David Romero Villamizar | Modelador ABM | `modelo/abm_campus.nlogo`, `docs/odd/` |
| Astrid Daniela Martinez Ruiz | SD, datos y experimentos | `modelo/sd_campus.nlogo`, `parametros/`, `experimentos/`, `analisis/` |
| Camilo Andrés Acosta Bernal  | Documentación y presentación | `docs/articulo/`, `docs/presentacion/`, `figuras/` |

## 11. Cómo citar

> Romero, J., Martinez, A. y Acosta, C. (2026). *Propagación de una enfermedad respiratoria en la comunidad universitaria de la Universidad de los Llanos: simulación basada en agentes y sistemas dinámicos*. Proyecto de Simulación Computacional, Universidad de los Llanos. Repositorio: _URL_.

## 12. Licencia

Código bajo licencia **MIT** (ver `LICENSE`). El texto del informe y las figuras quedan bajo **CC BY 4.0**, salvo indicación contraria. _(Ajusten si el docente o la universidad exigen otra.)_

## 13. Referencias principales

Las referencias completas están en `parametros/referencias.bib` y en el artículo. Puntos de partida:

- Kermack, W. O. y McKendrick, A. G. (1927). Modelo compartimental clásico.
- Wilensky, U. y Rand, W. (2015). *An Introduction to Agent-Based Modeling*. MIT Press.
- Grimm, V. et al. Protocolo ODD para describir modelos basados en agentes.
- Lauer, S. A. et al. (2020). Incubation period of COVID-19. *Ann Intern Med*. doi:10.7326/M20-0504
- Biggerstaff, M. et al. (2014). Estimates of the reproduction number for seasonal, pandemic, and zoonotic influenza. *BMC Infect Dis*.
