# Pr2B : Procedimiento

Empezamos declarando el contador de interrupciones con ```volatile```, lo que evitara que se elimine a causa de optimizaciones del compilador.
```volatile int interruptCounter;```

Declaramos también un contador para ver cuantas interrupciones han ocurrido desde el principio del programa, este no requiere ```volatile```, ya que solo vamos a usarlo en el loop principal (main loop). ```int totalInterruptCounter;```

Para configurar el timer, primero empezaremos cun un puntero ```hw_timer_t * timer = NULL;``` Y la última variable que tenemos que declarar es del tipo ```portMUX_TYPE``` , esta la usaremos para la sincronización entre el main loop y la ISR.
```
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;
```
Seguimos con un void:
```
void setup() {
Serial.begin(9600);
timer = timerBegin(0, 80, true);
timerAttachInterrupt(timer, &onTimer, true);
timerAlarmWrite(timer, 1000000, true);
timerAlarmEnable(timer);
}
```
Dentro de este loop inicimaos el timer con timerBegin . La funcion recibira como entrada el numero que queramos usar (en nuestro caso el 0), el valor del prescaler (en nuestro caso 80) e indicamos si el contador cuenta hacia adelante (true) o si lo hace hacia atrás (false). ```timer = timerBegin(0, 80, true);``` Ahora usaremos la función ```timerAttachInterrupt ```,  la funcion recibe como entrada un puntero, con direccion a onTimer la cual concretaremos y la utilizaremos para controlar las interrupciones. Por ultimo concretaremos la variable true para que sea tipo edge ```timerAttachInterrupt(timer, &onTimer, true);```. A continuación usaremos la función ```timerAlarmWrite``` , en esta concretaremos 3 valores: el puntero al temporizador, valor del contador en el que se genera la interrupción y por ultimo un indicador que determina si se genera automaticamente la interrupción.
```
timerAlarmWrite(timer, 1000000, true);
```
Finalmente llamamos a la función timerAlarmEnable , en esta pasamos como entrada nuestra variable de temporizador.
```
timerAlarmEnable(timer);
El main loop completo es el siguiente:
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
Como final nos queda un progama capaz de contar en numero total de interrupciones ( totalInterruptCounter ) i imprimirlo al puerto serie. Para acabar explicaremos como funciona la función ISR (interrupt service routine). Esta función nos imprime cuando ocurre un error y augmenta el numero maximo de interrupciones. Esto se produce dentro de una sección crítica, declarada con ```portENTER_CRITICAL_ISR``` y ```portEXIT_CRITICAL_ISR``` . El codigo del ISR es el siguiente:
```
void IRAM_ATTR onTimer() {
portENTER_CRITICAL_ISR(&timerMux);
interruptCounter++;
portEXIT_CRITICAL_ISR(&timerMux);
}
```
#### Funcionamiento de la impresión
El objetivo del programa es escribir por pantalla las veces que se interrumpe el programa. La impresión sera lo siguiente:
```
An intrrupt as occurred. Total number: 1
An intrrupt as occurred. Total number: 2
An intrrupt as occurred. Total number: 3
An intrrupt as occurred. Total number: 4
An intrrupt as occurred. Total number: 5
An intrrupt as occurred. Total number: 6
An intrrupt as occurred. Total number: 7
```
...
De manera infinita hasta nuevo aviso