# Ocultamiento y recuperación de mensajes mediante permutación espectral

Proyecto de **Análisis de Señales** · Universidad Militar Nueva Granada (UMNG) · RA 6.1

**Integrantes:**
- Laura Valentina Patino Olarte
- Salome Bohorquez Blanco
- Nicol Juliana Rodriguez Caicedo

##  Objetivo

Ocultar cinco mensajes de audio (**Secret**) reorganizando sus coeficientes complejos de Fourier según el orden de magnitudes de un audio de referencia (**Target**) (procedimiento **Imitation**), recuperarlos con el código **Reverse** proporcionado por la profesora y evaluar la calidad de la reconstrucción con **SNR** y **correlación de Pearson**.

## 📁 Estructura del repositorio

```
├── 01_Imitation.ipynb            # Imitation: permutación espectral, audios codificados y claves
├── 02_Reverse_FFT_GitHub.ipynb   # Reverse (código de la profesora, sin modificar)
├── 03_Metricas.ipynb             # SNR y correlación de Pearson
├── audios/                       # target.wav y secret1..5.wav (originales suministrados)
├── encoded/                      # encoded_1..5.wav
├── keys/                         # FFT_key_1..5.csv
├── recovered/                    # recovered_1..5.wav
├── imagenes/                     # gráficas usadas en este README
└── metricas.csv                  # tabla de resultados
```

## Cómo ejecutar

1. Abrir `01_Imitation.ipynb` y ejecutarlo completo. Genera `encoded/encoded_i.wav` y `keys/FFT_key_i.csv` para los cinco secretos.
2. El código Reverse de la profesora lee `encoded.wav` y `FFT_key.csv` con nombres fijos. Para cada secreto *i* (1 a 5): copiar `encoded/encoded_i.wav` como `encoded.wav` y `keys/FFT_key_i.csv` como `FFT_key.csv`, ejecutar `02_Reverse_FFT_GitHub.ipynb` y guardar la salida `recovered.wav` como `recovered/recovered_i.wav`.
3. Ejecutar `03_Metricas.ipynb`, que compara `audios/secret{i}.wav` con `recovered/recovered_{i}.wav` y escribe `metricas.csv`.

Dependencias: `numpy`, `scipy`, `matplotlib`, `pandas` (y `tabulate` para imprimir la tabla).

##  Método (Imitation)

Todos los audios están a 44 100 Hz, son mono y tienen 383 184 muestras (8,69 s). Para cada Secret:

1. Se calculan `rfft` del Secret (`S`) y del Target (`T`); cada coeficiente es un número complejo (magnitud y fase).
2. Se definen las posiciones permutables `P`: todas excepto el coeficiente DC y, si N es par, el de Nyquist, que permanecen fijos.
3. Se ordenan las posiciones de menor a mayor magnitud en ambas señales: `I1 = argsort(|S[P]|)`, `I2 = argsort(|T[P]|)`.
4. Se asocian las posiciones del mismo rango: el coeficiente de rango *r* del Secret se coloca donde el Target tiene rango *r*. La clave es `K[P[I2]] = P[I1]` (con `K` inicialmente la identidad).
5. Se reorganiza el espectro completo, `E = S[K]`, moviendo cada coeficiente complejo con su parte real e imaginaria juntas.
6. `irfft(E)` produce la señal **Encoded**, guardada en `float32` sin normalizar. La clave se guarda en `FFT_key_i.csv` con un encabezado de una línea (el Reverse usa `skiprows=1`).

El Reverse hace `X_rec[K] = X_enc`, la inversa exacta de `E = S[K]`.

## 📊 Resultados

| Mensaje secreto | SNR (dB) | Correlación de Pearson |
|---|---:|---:|
| Secret 1 | 152,42 | 1,0000 |
| Secret 2 | 152,21 | 1,0000 |
| Secret 3 | 152,41 | 1,0000 |
| Secret 4 | 152,37 | 1,0000 |
| Secret 5 | 152,26 | 1,0000 |

La correlación vale 1 hasta el duodécimo decimal en los cinco casos.

## Análisis

### Dominio temporal

![Señales originales](imagenes/Grafica_tiempo.png)

El Target tiene ráfagas de actividad separadas por pausas largas casi en silencio y su amplitud llega a unos 0,68. El Secret 1 es más continuo, con actividad sostenida entre aproximadamente 0,6 s y 8,7 s, pausas cortas y una amplitud máxima menor (≈ 0,4). Es decir, difieren en amplitud, en la cantidad y duración de las pausas y en la distribución de la actividad en el tiempo.

### Dominio espectral

![Espectros de magnitud](imagenes/Grafica_espectro.png)

En ambas señales la energía se concentra por debajo de ~2 kHz y el resto del espectro es mucho menor, como corresponde a voz. El pico principal del Target está en ≈ 639 Hz (magnitud ≈ 668), mientras que el del Secret 1 está en ≈ 135 Hz y es casi el doble de grande (≈ 1282). Los picos principales están, por tanto, en frecuencias distintas.

### Espectro reorganizado

![Espectro original y reorganizado](imagenes/Grafica_espectro_ordenado.png)

El espectro reorganizado del Secret 1 presenta sus picos en la zona de baja frecuencia donde el Target tiene los suyos, con su coeficiente mayor cerca de 640 Hz. Esto ocurre porque la permutación asigna posiciones por **rango**: el coeficiente de mayor magnitud del Secret va a la posición del mayor del Target, el segundo mayor a la del segundo mayor del Target, y así sucesivamente. Los valores son los mismos coeficientes del Secret (el máximo sigue siendo ≈ 1282, mayor que el del Target), pero sus posiciones siguen el patrón de magnitudes del Target.

### Señal codificada

![Señal codificada](imagenes/Grafica_encoded.png)

Al mover los coeficientes, cada uno queda en una frecuencia distinta a la original con su fase, así que se pierde la relación entre fases que determinaba cuándo ocurría cada sonido. La señal Encoded tiene actividad distribuida en toda su duración, sin las pausas del original, y se percibe como ruido.

### Calidad de la recuperación

![SNR por mensaje](imagenes/SNR_por_mensaje.png)

- **Mejor recuperación:** Secret 1, con 152,42 dB. Le siguen de cerca Secret 3 (152,41), Secret 4 (152,37) y Secret 5 (152,26), y el más bajo es Secret 2 (152,21).
- **¿Son similares los cinco?** Sí. La diferencia entre el mejor y el peor es de unos 0,2 dB, por lo que ninguno presenta una diferencia considerable. Con una variación tan pequeña no se puede atribuir a una causa clara; es coherente con ruido numérico de la misma naturaleza en los cinco casos, cuyo nivel relativo depende ligeramente de cada señal.
- **SNR:** valores en torno a 152 dB significan que la energía del error es del orden de 10⁻¹⁵ veces la de la señal. El error existe y es finito, pero es muchísimo menor que cualquier diferencia audible.
- **Pearson:** vale 1 en los cinco casos, es decir, la forma de onda recuperada es prácticamente idéntica a la original. Esta métrica compara forma (ignora escala y desplazamiento) y casi no distingue entre los cinco audios, mientras que el SNR sí permite ver las pequeñas diferencias de error. Por eso se interpretan ambas.

### ¿Por qué una permutación reversible tiene errores al guardar y cargar?

La permutación en sí es exactamente reversible: cada coeficiente solo cambia de posición, y `X_rec[K] = X_enc` devuelve cada uno a su lugar. El error aparece porque el audio se almacena con precisión finita. El Encoded se guarda en `float32` (24 bits de mantisa), por lo que cada muestra se redondea. Esos redondeos entran en la `rfft` del Reverse, se propagan a todos los coeficientes y, al aplicar `irfft`, quedan repartidos como un pequeño error en la señal recuperada, que además se guarda otra vez en `float32`. El orden de magnitud del error es el esperado de la precisión de `float32`. Si se hubiera guardado en un formato de menor resolución (por ejemplo, enteros de 16 bits) o se hubiera normalizado, el error sería mucho mayor.

##  Conclusiones

- La permutación espectral de coeficientes complejos oculta los cinco mensajes (el audio codificado suena como ruido) y permite recuperarlos con el Reverse de la profesora con SNR ≈ 152 dB y correlación 1.
- Los cinco mensajes se recuperan con calidad prácticamente igual; el error restante se debe a la precisión finita del almacenamiento, no a la permutación.
- Conservar la parte real e imaginaria de cada coeficiente es lo que hace posible la recuperación sin guardar la fase por separado.

##  Créditos

El código Reverse (`02_Reverse_FFT_GitHub.ipynb`) fue suministrado por la profesora Dora María Ballesteros Larrotta, del libro *Análisis de Señales Utilizando Python* (UMNG), y se incluye sin modificaciones.
