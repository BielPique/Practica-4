# PRÁCTICA 4: SISTEMAS OPERATIVOS EN TIEMPO REAL

## EJERCICIO PRÁCTICO 1

En este ejercicio hemos trabajado con un código que crea dos tareas donde se usa FreeRTOS, un sistema operativo de tiempo real.
```
#include <Arduino.h>

void anotherTask(void * parameter) {
  for(;;) { 
    Serial.println("this is another Task");
    delay(1000);
  }
  vTaskDelete(NULL);  // Esta línea nunca se ejecutará, pero es buena práctica incluirla
}

void setup() {
  Serial.begin(115200);

  xTaskCreate(
    anotherTask,  // Función de la tarea
    "another Task",  // Nombre de la tarea
    10000,  // Tamaño de la pila en bytes
    NULL,  // Parámetro de la tarea
    1,  // Prioridad de la tarea
    NULL  // Manejador de la tarea
  );
}

void loop() {
  Serial.println("this is ESP32 Task");
  delay(1000);
}
```

La primera tarea se ejecuta en el loop(), y saca por pantalla repetidamente la frase (“this is ESP32 Task”) con un delay de 1000 milisegundos (1 segundo) entre ejecuciones.

La segunda tarea tiene la misma finalidad que la primera con la diferencia de que está creada en una función llamada setup() y que por pantalla imprime (“this is another task”).

Las dos tareas funcionan en simultáneo y una vez compilado el código y ejecutado, el serial monitor nos muestra lo siguiente:


![image](https://github.com/user-attachments/assets/5cab894e-f6dd-404c-b2b5-f002396fc1aa)

## EJERCICIO PRÁCTICO 2

```
#include <Arduino.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"

const int ledPin = 2;  // Pin del LED
SemaphoreHandle_t xSemaphore;  // Semáforo para sincronizar las tareas

// Tarea para encender el LED
void TaskTurnOn(void *parameter) {
  for (;;) {
    if (xSemaphoreTake(xSemaphore, portMAX_DELAY)) { // Toma el semáforo
      digitalWrite(ledPin, HIGH);
      Serial.println("LED ENCENDIDO");
      delay(1000);  // Espera un segundo antes de liberar el semáforo
      xSemaphoreGive(xSemaphore); // Libera el semáforo
    }
    vTaskDelay(100 / portTICK_PERIOD_MS);  // Pequeña espera antes de intentar tomar el semáforo nuevamente
  }
}

// Tarea para apagar el LED
void TaskTurnOff(void *parameter) {
  for (;;) {
    if (xSemaphoreTake(xSemaphore, portMAX_DELAY)) { // Toma el semáforo
      digitalWrite(ledPin, LOW);
      Serial.println("LED APAGADO");
      delay(1000);  // Espera un segundo antes de liberar el semáforo
      xSemaphoreGive(xSemaphore); // Libera el semáforo
    }
    vTaskDelay(100 / portTICK_PERIOD_MS);
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);

  xSemaphore = xSemaphoreCreateBinary();  // Crear el semáforo

  xTaskCreate(TaskTurnOn, "Encender LED", 1000, NULL, 1, NULL);
  xTaskCreate(TaskTurnOff, "Apagar LED", 1000, NULL, 1, NULL);

  xSemaphoreGive(xSemaphore); // Inicializa el semáforo en "libre"
}

void loop() {
  // No se necesita código en loop(), ya que todo ocurre en las tareas de FreeRTOS
}
```

En este ejercicio hemos generado un código que intercala dos tareas y realicen la función de una especie de “semáforo”. Esto lo hemos logrado creando una tarea que encienda un LED que hemos conectado en el PIN 2 de nuestro microprocesador. Una vez encendido el LED imprimirá por pantalla (“LED ENCENDIDO”).
La segunda función se ejecuta con un delay de 1 segundo después de la anterior y apaga el LED junto con un mensaje en el serial monitor (“LED APAGADO”).

Al ejecutarlo se muestra lo siguiente:

![image](https://github.com/user-attachments/assets/52022675-ac74-404d-aaf4-47e9df8fe7f1)
