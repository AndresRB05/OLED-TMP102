# OLED-TMP102
Andres Felipe Rodriguez - Juan Sebastian Acuña 

#include "mbed.h"
#include "Adafruit_GFX.h"
#include "Adafruit_SSD1306.h"
#include <cstring>

#define tiempo_muestreo   1s
#define TMP102_ADDRESS 0x90 // Dirección del TMP102

//Pines y puertos 
BufferedSerial serial(USBTX, USBRX);
I2C i2c(D14,D15);
Adafruit_SSD1306_I2c oled(i2c, D0);
AnalogIn ain(A0);

// Variables globales 
char comando[3] = {0x01, 0x60, 0xA0};
char data[2];
char men[40];
const char *mensaje_inicio = "Arranque del programa\n\r";

// Configuración de la pantalla OLED
void setupOLED() {
    oled.begin();
    oled.setTextSize(1);
    oled.setTextColor(1);
    oled.display();
    ThisThread::sleep_for(3000ms);
    oled.clearDisplay();
    oled.display();
    oled.printf("Test\r\n");
    oled.display();
}

// Lectura y conversión del voltaje del sensor analógico
void obtenerVoltaje(float &Vin, int &ent, int &dec) {
    Vin = ain * 3.3;
    ent = static_cast<int>(Vin);
    dec = static_cast<int>((Vin - ent) * 10000);
    sprintf(men, "El voltaje es:\n\r %01u.%04u volts \n\r", ent, dec);
}

// Leer temperatura desde el sensor TMP102
void obtenerTemperatura(int &ent, int &dec) {
    comando[0] = 0; // Registro de temperatura
    i2c.write(TMP102_ADDRESS, comando, 1); // Enviar el comando para leer
    i2c.read(TMP102_ADDRESS, data, 2); // Leer 2 bytes
    int16_t temp = (data[0] << 4) | (data[1] >> 4);
    float Temperatura = temp * 0.0625;
    ent = static_cast<int>(Temperatura);
    dec = static_cast<int>((Temperatura - ent) * 10000);
    sprintf(men, "La Temperatura es:\n\r %01u.%04u Celsius\n\r", ent, dec);
}

// Actualizar pantalla OLED con nueva información
void mostrarEnOLED() {
    oled.clearDisplay();
    oled.setTextCursor(0, 2);
    oled.printf(men);
    oled.display();
}

// Enviar datos por puerto serial
void mostrarEnSerial() {
    serial.write(men, strlen(men));
}

int main() {
    setupOLED();
    serial.write(mensaje_inicio, strlen(mensaje_inicio));
    
 while (true) {
        float Vin = 0.0;
        int ent = 0, dec = 0;

 // Lectura del voltaje y despliegue
        obtenerVoltaje(Vin, ent, dec);
        mostrarEnOLED();
        mostrarEnSerial();

 // Lectura de la temperatura y despliegue
        obtenerTemperatura(ent, dec);
        mostrarEnOLED();
        mostrarEnSerial();

ThisThread::sleep_for(tiempo_muestreo);
    }
}
