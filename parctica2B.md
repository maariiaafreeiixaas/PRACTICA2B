# Pràctica 2

## Pràctica B: Interrupció per Timer

### Objectiu
Utilitzar un temporitzador per generar interrupcions periòdiques i comptar el nombre total d'interrupcions.

### Codi
El codi següent configura un temporitzador que genera una interrupció cada segon.

```cpp
volatile int interruptCounter;
int totalInterruptCounter;
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

//ISR: La funció onTimer() incrementa un comptador d'interrupcions utilitzant una secció crítica
void IRAM_ATTR onTimer() 
{
 portENTER_CRITICAL_ISR(&timerMux);
 interruptCounter++;
 portEXIT_CRITICAL_ISR(&timerMux);
}

//Iniciació: La funció setup() inicialitza el temporitzador amb un prescaler de 80 (perquè la freqüència del temporitzador sigui 1 MHz) i configura una alarma que s'activa cada segon.
void setup() 
{
 Serial.begin(115200);
 timer = timerBegin(0, 80, true);
 timerAttachInterrupt(timer, &onTimer, true);
 timerAlarmWrite(timer, 1000000, true);
 timerAlarmEnable(timer);
}

//Bucle principal: La funció loop() comprova si hi ha interrupcions pendents, decrementa el comptador d'interrupcions i incrementa el comptador total d'interrupcions que despres treu pel port serie
void loop() 
{
 if (interruptCounter > 0) {
 portENTER_CRITICAL(&timerMux);
 interruptCounter--;
 portEXIT_CRITICAL(&timerMux);
 totalInterruptCounter++;
 Serial.print("An interrupt as occurred. Total number: ");
 Serial.println(totalInterruptCounter);
 }
}

Sortides a través de la impressió sèrie:

An interrupt has occurred. Total number: 1
An interrupt has occurred. Total number: 2
An interrupt has occurred. Total number: 3
…