# Practica2-A
Participants: Alexandre Pascual / Marti Vila
## Interrupción de Botón en ESP32

Este proyecto configura un **botón con interrupción** en un **ESP32** para contar cuántas veces se ha presionado. Además, desactiva la interrupción después de 1 minuto.

### Descripción del Código

El código utiliza una **interrupción de hardware** para detectar cuando el botón es presionado. Se usa un **struct** para almacenar la información del botón, y un mecanismo para **deshabilitar la interrupción tras 1 minuto**.

### Funcionamiento
1. Se configura un botón en el **pin 18** con **resistencia pull-up interna**.
2. Cuando el botón se presiona, una interrupción incrementa el contador de pulsaciones.
3. En el `loop()`, se imprime el número total de veces que se ha presionado el botón.
4. **Después de 1 minuto**, la interrupción se desactiva para ahorrar recursos.

#### Estructura `Button`
```cpp
struct Button {
    const uint8_t PIN;
    uint32_t numberKeyPresses;
    bool pressed;
};
```
- **`PIN`**: Almacena el número de pin donde está conectado el botón.
- **`numberKeyPresses`**: Cuenta cuántas veces se ha presionado.
- **`pressed`**: Indica si el botón está actualmente presionado.

#### Configuración del Botón e Interrupción (`setup()`)
```cpp
void setup() {
    Serial.begin(115200);
    pinMode(button1.PIN, INPUT_PULLUP);
    attachInterrupt(button1.PIN, isr, FALLING);
}
```
- Se inicia la comunicación serial.
- Se configura el **pin 18 como entrada pull-up**.
- Se **adjunta la interrupción** al botón, que se activa en un **flanco de bajada** (`FALLING`).

#### Función de Interrupción (`isr()`)
```cpp
void IRAM_ATTR isr() {
    button1.numberKeyPresses += 1;
    button1.pressed = true;
}
```
- Se ejecuta cuando se detecta una interrupción.
- Incrementa el contador de pulsaciones.
- Marca `pressed` como `true` para indicar que el botón ha sido presionado.

#### Bucle Principal (`loop()`)
```cpp
void loop() {
    if (button1.pressed) {
        Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
        button1.pressed = false;
    }
    
    // Desactivar la interrupción después de 1 minuto
    static uint32_t lastMillis = 0;
    if (millis() - lastMillis > 60000) {
        lastMillis = millis();
        detachInterrupt(button1.PIN);
        Serial.println("Interrupt Detached!");
    }
}
```
- Si `button1.pressed` es `true`, se imprime el número total de veces que se ha presionado.
- Usa `millis()` para **deshabilitar la interrupción después de 1 minuto**.


### Notas
- `IRAM_ATTR` asegura que la interrupción se ejecuta desde la **RAM interna** para mejorar la estabilidad.
- `detachInterrupt()` evita que el ESP32 siga respondiendo a eventos del botón después de 1 minuto.

### Salida esperada en Serial Monitor
```
Button 1 has been pressed 1 times
Button 1 has been pressed 2 times
...
Interrupt Detached!
```


