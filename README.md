# Nowcast mensual de la inflación argentina (IPC)

Pronóstico del IPC mensual del INDEC **después del REM y antes del dato oficial**, con intervalo del 80%.
Proyecto personal de aprendizaje: lo programé yo paso a paso, así que el código es simple a propósito.

## Qué hace
Cada mes, entre la publicación del REM (4° día hábil) y la del INDEC, pronostica la inflación del mes y publica el número antes de conocer el dato.

**Modelos** (promedio simple de los tres):
1. ElasticNetCV sobre el nivel de inflación.
2. ElasticNetCV sobre el *error del REM* (aprende cuánto se equivoca el REM; pronóstico = REM + corrección).
3. Random Forest sobre el error del REM.

**Features:** inflación rezagada, REM del mes, variación del tipo de cambio mayorista, BADLAR y reservas (BCRA), estacionalidad.
**Benchmarks:** REM y caminata aleatoria.
**Validación:** backtest *expanding window* desde 2022-01; en cada mes se entrena solo con meses anteriores.

## Resultados (backtest, RMSE en p.p.)
| Muestra | EN | EN s/REM | RF s/REM | Promedio | REM |
|---|---|---|---|---|---|
| 2022-01 en adelante (57 meses) | 0,70 | 0,67 | 0,64 | 0,65 | 0,75 |
| 2024-05 en adelante (29 meses) | 0,32 | 0,29 | 0,29 | **0,27** | 0,39 |

**Honestidad estadística:** la mejora contra el REM **no es estadísticamente significativa** (Diebold-Mariano simple, t ≈ 1,5; con errores HAC sería menor). Con ~30-60 meses y un régimen cambiante hay poca potencia. Los cortes de evaluación se fijaron antes de ver los resultados, pero hay grados de libertad de investigador (ventana, features, modelos) y no corrijo por múltiples comparaciones.

## Intervalos
Conformal con ventana de 24 meses y ajuste adaptativo (ACI, alpha=0,2, gamma=0,05). Probé primero errores absolutos y después errores **relativos al nivel pronosticado** (versión normalizada), porque el ancho no se adaptaba a la volatilidad. Ese cambio fue posterior a ver resultados (post-hoc); reporto ambas.
Cobertura observada desde 2024-05 con la versión normalizada: **76%** (nominal 80%), ancho medio 0,79 p.p.

## Cómo correrlo cada mes
1. Esperar el REM del mes (el Excel de tablas sale el mismo día).
2. Abrir `notebooks/02_pronostico_mensual.ipynb` y correr todo (con `BAJAR = True` descarga los datos del BCRA).
3. El notebook agrega una fila a `data/pronosticos.csv` (nunca pisa un mes ya registrado), compara contra lo observado y guarda `figs/nowcast_<mes>.png`.
4. Cuando sale el dato del INDEC, se actualiza el registro y se mide el error.

`data/pronosticos.csv` es el registro de lo publicado antes del dato, para que nadie (yo incluido) pueda ajustar el pasado.

## Estructura
```
notebooks/01_bcra_inflacion.ipynb     exploración y desarrollo
notebooks/02_pronostico_mensual.ipynb pipeline mensual
data/raw/  data/processed/  data/pronosticos.csv
figs/
```

## Limitaciones
- Muestra corta (desde 2019) y cambios de régimen: los resultados pueden no extrapolarse.
- El REM ya contiene mucha información; el valor agregado es chico.
- No se usan datos de alta frecuencia (precios online, alimentos semanales).

## Trabajo relacionado
Forte (2024), BBVA Research WP 24/10: random forest para inflación argentina, con precisión similar al REM.

## Ideas para v2
Brecha cambiaria como feature, promedio ponderado por desempeño, LightGBM / Extra Trees, autocorrelación de los errores, error por horizonte.

## Datos
BCRA (API de estadísticas v4 y REM), INDEC. No hay credenciales en el repo.
