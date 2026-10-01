# TouchDesigner → Strudel mediante MIDI

Este experimento prueba la comunicación de parámetros desde **TouchDesigner** hacia **Strudel** utilizando MIDI.

La comunicación ocurre dentro del mismo computador Windows mediante un puerto MIDI virtual creado con **loopMIDI**.

```text
TouchDesigner
     │
     │ MIDI Control Change
     ▼
 loopMIDI Port
     │
     ▼
   Strudel
```

El objetivo inicial es enviar dos valores desde TouchDesigner:

```text
Channel 1, CC 0
Channel 1, CC 1
```

y utilizarlos en Strudel para controlar:

```text
CC 0 → gain
CC 1 → filter cutoff
```

---

## 1. Crear el puerto virtual con loopMIDI

Instalar y abrir **loopMIDI**.

Crear un puerto llamado exactamente:

```text
loopMIDI Port
```

<img width="482" height="279" alt="image" src="https://github.com/user-attachments/assets/53c0b209-10aa-4654-9be9-3516f0eb552d" />



Este puerto funciona como un cable MIDI virtual entre aplicaciones.

```text
TouchDesigner
      │
      ▼
loopMIDI Port
      │
      ▼
Strudel
```

loopMIDI debe estar ejecutándose mientras se realiza el experimento.

---

## 2. MIDI Device Mapper en TouchDesigner

Abrir:

```text
Dialogs → MIDI Device Mapper
```

Crear un `Device Mapping` con:

```text
ID          1
In Device   loopMIDI Port
Out Device  loopMIDI Port
MIDI Map    none
Channel     1
```

El resultado debe ser equivalente a:

```text
ID  In Device       Out Device      MIDI Map  Ch
1   loopMIDI Port   loopMIDI Port   none      1
```

<img width="597" height="362" alt="image" src="https://github.com/user-attachments/assets/eb0090d5-3cc3-4a81-9ad6-c5e648b0fb6a" />



TouchDesigner guarda esta configuración dentro del proyecto mediante:

```text
/local/midi
```


<img width="1208" height="609" alt="image" src="https://github.com/user-attachments/assets/9ff8631b-4a81-4de9-9224-d577d6bcc74d" />




La tabla principal de dispositivos se encuentra en:

```text
/local/midi/device
```

En este experimento contiene aproximadamente:

```text
id | indevice      | outdevice     | definition | channel
1  | loopMIDI Port | loopMIDI Port |            | 1
```

Los mapas MIDI personalizados del proyecto se almacenan bajo:

```text
/local/midi/userdevices
```

Por tanto, la configuración MIDI forma parte del `.toe`.

Lo único externo al proyecto es la existencia del dispositivo:

```text
loopMIDI Port
```

que debe crearse previamente en Windows.

---

## 3. Convención MIDI utilizada

En este experimento se utiliza:

```text
MIDI Channels : 1 ... 16
Control Change: 0 ... 127
```

Es importante distinguir ambas numeraciones.

El canal MIDI se mantiene numerado desde `1`:

```text
Channel 1
Channel 2
...
Channel 16
```

Los números de Control Change se manejan desde `0`:

```text
CC 0
CC 1
...
CC 127
```

En TouchDesigner se configura:

```text
1 Based Index = Off
```

<img width="597" height="342" alt="image" src="https://github.com/user-attachments/assets/098aa447-9924-4d5a-91ae-758f7a75f573" />

para que el número utilizado en el nombre del canal coincida directamente con el número real de CC.

Así:

```text
TouchDesigner      MIDI

ch1c0           →  Channel 1, CC 0
ch1c1           →  Channel 1, CC 1
ch1c2           →  Channel 1, CC 2
```

No se utiliza `ch0`.

Los canales MIDI continúan siendo:

```text
1 ... 16
```

---

## 4. MIDI Out CHOP

Crear un `MIDI Out CHOP`.

Configurarlo así:

```text
Active           On
MIDI Destination Device
Device Table     /local/midi/device
Device ID        1
1 Based Index    Off
Channel Prefix   ch
```

El `Device ID 1` corresponde al mapping creado previamente:

```text
ID 1 → loopMIDI Port
```

Por tanto, el `MIDI Out CHOP` no necesita conocer directamente el nombre `loopMIDI Port`.

Utiliza la entrada:

```text
/local/midi/device
       │
       └── ID 1
              │
              └── loopMIDI Port
```

Esto desacopla la red de TouchDesigner del dispositivo MIDI físico o virtual utilizado.

---

## 5. Enviar Control Change desde TouchDesigner

Para la prueba se crean dos canales:

```text
ch1c0 = 0.42
ch1c1 = 0.39
```

Por ejemplo:

```text
Constant CHOP          Constant CHOP
ch1c0 = 0.42           ch1c1 = 0.39
      │                     │
      └────────┐   ┌────────┘
               ▼   ▼
             Merge CHOP
                 │
                 ▼
           MIDI Out CHOP
                 │
                 ▼
          loopMIDI Port
```

Los nombres tienen significado:

```text
ch1c0
│ │ └─ CC 0
│ └─── controller
└───── MIDI Channel 1
```

y:

```text
ch1c1
│ │ └─ CC 1
│ └─── controller
└───── MIDI Channel 1
```

Por tanto:

```text
ch1c0 = 0.42 → Channel 1, CC 0
ch1c1 = 0.39 → Channel 1, CC 1
```

---

## 6. Valores normalizados y valores MIDI

En la red de TouchDesigner trabajamos cómodamente con valores normalizados:

```text
0.0 ... 1.0
```

MIDI Control Change utiliza tradicionalmente valores enteros:

```text
0 ... 127
```

Por tanto:

```text
0.00 →   0
0.50 → ~64
1.00 → 127
```

En nuestra prueba:

```text
0.42 × 127 ≈ 53
0.39 × 127 ≈ 50
```

Por eso un `MIDI In CHOP` que recibe estos mensajes puede mostrar aproximadamente:

```text
53  ch1ctrl0
50  ch1ctrl1
```

aunque originalmente se hubieran generado como:

```text
0.42 ch1c0
0.39 ch1c1
```

---

## 7. MIDI In CHOP

También puede utilizarse un `MIDI In CHOP` para comprobar qué está circulando por el puerto.

Configurarlo así:

```text
Active           On
Mode             Automatic
MIDI Source      Device
Device Table     /local/midi/device
Device ID        1
1 Based Index    Off
Controller Format 7 bit Controllers
```


<img width="783" height="536" alt="image" src="https://github.com/user-attachments/assets/b288c80a-3ee3-4d5f-bb11-f4ee97364b1c" />


Al utilizar el mismo:

```text
Device ID = 1
```

el `MIDI In CHOP` escucha:

```text
loopMIDI Port
```

Si TouchDesigner envía:

```text
ch1c0 = 0.42
ch1c1 = 0.39
```

es posible observar:

```text
53 ch1ctrl0
50 ch1ctrl1
```

Esto confirma que los mensajes realmente están atravesando el puerto MIDI.

Como `loopMIDI Port` está configurado tanto como entrada como salida, TouchDesigner también puede recibir los mensajes que él mismo está enviando. Esto resulta útil para depuración.

---

## 8. Modelo mental del mensaje

Para este experimento, un mensaje MIDI Control Change puede pensarse como:

```text
Channel + CC + Value
```

Por ejemplo:

```text
Channel = 1
CC      = 0
Value   = 53
```

conceptualmente significa:

```text
En Channel 1,
el controlador CC 0
ahora tiene el valor 53.
```

TouchDesigner genera este mensaje a partir de:

```text
ch1c0 = 0.42
```

La cadena completa es:

```text
0.42
 │
 ▼
ch1c0
 │
 ▼
MIDI Out CHOP
 │
 ▼
Channel 1
CC 0
Value 53
 │
 ▼
loopMIDI Port
 │
 ▼
Strudel
```

---

## 9. Recibir MIDI en Strudel

Strudel puede abrir el puerto MIDI mediante:

```javascript
const midi = await midin('loopMIDI Port')
```

Después podemos leer un Control Change indicando:

```javascript
midi(cc, channel)
```

Para este experimento:

```javascript
const cc0 = midi(0, 1)
const cc1 = midi(1, 1)
```

corresponde a:

```text
TouchDesigner       Strudel

ch1c0            →  midi(0, 1)
ch1c1            →  midi(1, 1)
```

Es decir:

```text
Channel 1, CC 0 → cc0
Channel 1, CC 1 → cc1
```

---

## 10. Código de Strudel

El código utilizado para la prueba es:

```javascript
setcpm(100)

const midi = await midin('loopMIDI Port')

const cc0 = midi(0, 1) // Channel 1, CC 0
const cc1 = midi(1, 1) // Channel 1, CC 1

$: note("c3 e3 g3 c4")
  .sound("sawtooth")
  .legato(1)
  .lpf(cc1.range(100, 5000))
  .gain(cc0)
```

Aquí:

```text
CC 0 → gain
CC 1 → low-pass filter cutoff
```

El valor recibido mediante:

```javascript
midi(0, 1)
```

puede utilizarse directamente como señal de control.

Mientras que:

```javascript
midi(1, 1)
```

se transforma desde su rango normalizado:

```text
0 ... 1
```

al rango:

```text
100 ... 5000 Hz
```

mediante:

```javascript
cc1.range(100, 5000)
```

Por tanto:

```text
TouchDesigner                Strudel

ch1c0 ──────────────────────► gain
 0..1                         0..1

ch1c1 ──────────────────────► lpf
 0..1                         100..5000 Hz
```

---

## 11. ¿Por qué especificar el canal en Strudel?

También es posible escribir:

```javascript
midi(0)
```

pero en ese caso no se está restringiendo explícitamente la entrada a un canal MIDI particular.

Por ejemplo, tanto:

```text
ch1c0
```

como:

```text
ch13c0
```

pueden afectar a una consulta de `CC 0` sin filtrado por canal.

Para evitar ambigüedades en este experimento usamos siempre:

```javascript
midi(cc, channel)
```

Por ejemplo:

```javascript
midi(0, 1)
```

significa explícitamente:

```text
Channel 1, CC 0
```

Esto permite utilizar los canales MIDI como espacios independientes de control.

Por ejemplo:

```text
Channel 1  → controles globales
Channel 2  → instrumento A
Channel 3  → instrumento B
```

y dentro de cada canal reutilizar:

```text
CC 0
CC 1
CC 2
...
```

---

## 12. Arquitectura final del experimento

La arquitectura completa queda:

```text
┌──────────────────────── TOUCHDESIGNER ────────────────────────┐

 Constant CHOP                     Constant CHOP
 ch1c0 = 0.42                      ch1c1 = 0.39
      │                                  │
      └───────────────┐  ┌───────────────┘
                      ▼  ▼
                   Merge CHOP
                       │
                       ▼
                  MIDI Out CHOP
                       │
                 Device ID = 1
                       │
                       ▼
                /local/midi/device
                       │
                       ▼
                 loopMIDI Port

└────────────────────────────┬───────────────────────────────────┘
                             │
                             │ MIDI
                             ▼
                     ┌──────────────┐
                     │   loopMIDI   │
                     │     Port     │
                     └──────┬───────┘
                            │
                            ▼
┌────────────────────────── STRUDEL ─────────────────────────────┐

                  await midin('loopMIDI Port')
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
            midi(0, 1)             midi(1, 1)
                 │                     │
                 ▼                     ▼
               gain                   lpf

└────────────────────────────────────────────────────────────────┘
```

---

## 13. Qué pertenece al `.toe` y qué no

La configuración puede entenderse en dos capas.

### Dentro de TouchDesigner

El `.toe` conserva la configuración del proyecto, incluyendo la información utilizada por:

```text
/local/midi
```

y el Device Mapper.

Por ejemplo:

```text
/local/midi/device
/local/midi/userdevices
```

Los MIDI In y MIDI Out CHOP pueden hacer referencia al mapper mediante:

```text
Device Table = /local/midi/device
Device ID    = 1
```

Esto evita configurar el nombre del dispositivo individualmente en cada operador.

### Fuera de TouchDesigner

El puerto:

```text
loopMIDI Port
```

pertenece a Windows y no forma parte del `.toe`.

Por tanto, para ejecutar este proyecto en otro computador es necesario:

1. instalar loopMIDI;
2. crear un puerto llamado `loopMIDI Port`;
3. abrir el `.toe`;
4. comprobar el MIDI Device Mapper;
5. abrir Strudel y ejecutar el código.

---

## 14. Resumen

La correspondencia utilizada en este experimento es:

```text
TouchDesigner     MIDI                  Strudel

ch1c0          →  Channel 1, CC 0   →  midi(0, 1)
ch1c1          →  Channel 1, CC 1   →  midi(1, 1)
```

y los datos recorren:

```text
TouchDesigner
      │
      ▼
MIDI Out CHOP
      │
      ▼
Device Mapper
      │
      ▼
loopMIDI Port
      │
      ▼
Strudel
      │
      ├── CC 0 → gain
      │
      └── CC 1 → filter cutoff
```

La idea central es que MIDI no transmite audio.

MIDI transmite **mensajes de control**:

```text
Channel + Controller + Value
```

y permite que dos aplicaciones independientes compartan parámetros mediante un protocolo estándar.
