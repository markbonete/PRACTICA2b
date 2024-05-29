# PRACTICA 2 - Part B

## Codi sencer
```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  interruptCounter++;
  portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {
  Serial.begin(115200);

  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, 1000000, true);
  timerAlarmEnable(timer);
}

void loop() {
  if (interruptCounter > 0) {
    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);

    totalInterruptCounter++;

    Serial.print("An interrupt as occurred. Total number: ");
    Serial.println(totalInterruptCounter);
  }
}
```

## Explicació del codi per parts
### Definició de les variables
```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

```
- interruptCounter: Variable que s'incrementa en la rutina d'interrupció. Es volatil degut a que es modifica en cualsevol moment i el compilador no ha d'optimitzarla.
- totalInterruptCounter: Comptador del total d'interrupcions gestionades.
- timer: Un punter a l'estructura hw_timer_t que representa el temporitzador de hardware.
- timerMux: Variable per gestionar la sincronització en la rutina d'interrupció.

### Rutina d' interrupció
```cpp
void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  interruptCounter++;
  portEXIT_CRITICAL_ISR(&timerMux);
}
```
La funció onTimer() és una funció que es col·loca a la memòria IRAM(per a que s'executi més ràpid).Cada vegada que es produiexi una interrupció en el temporitzador,la funció entrarà en la secció crítica(deshabilita temporalment les interrupcions per evitar interrupcions en el codi).
El comptador interruptCounter s'incrementarà i sortirà de la secció crítica.

### Setup
```cpp
void setup() {
  Serial.begin(115200);

  timer = timerBegin(0, 80, true);
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, 1000000, true);
  timerAlarmEnable(timer);
}
```
S'estableix la configuració inicial del programa:
- Serial.begin(115200): Inicialitza la comunicació sèrie en 115200 bauds.
- timer: Configura el temporitzador 0 amb un prescaler de 80 (cada tick del temporitzador correspon a  1 ms) i en mode de increment (true).
- timerattachInterrupt: Adjunta la rutina d'interrupció onTimer al temporitzador.
- timerAlarmWrite: Configura el temporitzador per generar una interrupció cada 1,000,000 ticks (1 segon) i reinicia el temporitzador automàticament.
- timerAlarmEnable: Habilita l'alarma del temporitzador i comenci a comptar.
  
### Loop
```cpp
void loop() {
  if (interruptCounter > 0) {
    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);

    totalInterruptCounter++;

    Serial.print("An interrupt as occurred. Total number: ");
    Serial.println(totalInterruptCounter);
  }
}
```
En cada iteració verifica si interruptCounter es major a 0. Si es el cas,entra en una secció crítica per decrementar la variable interruptCounter de forma segura. 
Un cop decrementada surt de la secció crítica.
Quan s'ha produit aquesta interrupció, incrementa el contador totalInterruptCounter .
Ens mostra per pantalla el número total d'interrupcions que hagi hagut.
