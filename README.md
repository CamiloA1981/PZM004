// PZM004
//Programa para lectura de vatimetro, las lecturas seran realizadas sobre potencia activa, reactica y factor de potencia.

#include <Arduino.h>
#include <ModbusMaster.h>
#include <SoftwareSerial.h>     // Si usas pines GPIO para RS485

// Definiciones para ESP32 LoRa V3 y RS485
const int ledPin = 2;       // Pin del LED (puedes ajustarlo)
const int RS485_TX_PIN = 17; // Pin de transmisión RS485 (TX)
const int RS485_RX_PIN = 16; // Pin de recepción RS485 (RX)
const int RS485_DE_RE = 4;   // Pin de habilitación de transmisión/recepción (DE/RE)

// Instancia de SoftwareSerial si usas pines GPIO
SoftwareSerial rs485Serial(RS485_RX_PIN, RS485_TX_PIN); // RX, TX

ModbusMaster node;
String cadena = "";

void preTransmission() {
  digitalWrite(RS485_DE_RE, HIGH); // Habilita la transmisión
}

void postTransmission() {
  digitalWrite(RS485_DE_RE, LOW);  // Deshabilita la transmisión
}

void setup() {
  Serial.begin(115200);           // Monitor serial a 115200 (mayor velocidad para ESP32)
  rs485Serial.begin(4800);        // Inicializa el puerto serial para RS485
  rs485Serial.setTimeout(1000);

  pinMode(ledPin, OUTPUT);
  pinMode(RS485_DE_RE, OUTPUT);

  digitalWrite(ledPin, LOW);
  digitalWrite(RS485_DE_RE, LOW); // Inicialmente en recepción

  node.begin(1, rs485Serial); // Usa SoftwareSerial para Modbus
  node.preTransmission(preTransmission);
  node.postTransmission(postTransmission);
}

void loop() {
  Lectura_RX();
  digitalWrite(ledPin, HIGH);
  delay(1000);
  Solicitar();
  digitalWrite(ledPin, LOW);
  delay(1000);
}

void Solicitar() {
  // Datos del mensaje Modbus (sin cambios)
  int Inicial_Byte1 = 0xFF;
  // ... (resto de las definiciones)

  preTransmission(); // Habilita la transmisión

  Serial.println("Radiacion");
  rs485Serial.write(Inicial_Byte1);
  // ... (resto de los Serial1.write)

  postTransmission(); // Deshabilita la transmisión
  delay(1000);
}

void Lectura_RX() {
  if (rs485Serial.available()) {
    while (rs485Serial.available()) {
      char data = rs485Serial.read();
      cadena += data;
    }
    Serial.println(cadena);
    //Serial1.println(cadena); // No es necesario retransmitir a RS485 en la mayoria de los casos
    cadena = "";
  }
}
