# AutoWin
Código para automação de janela residencia  com comando por aplicativo mobile

#include <ESP32Servo.h> //Ativação do servo motor
#include <WiFi.h>
#include <IOXhop_FirebaseESP32.h>
#include <ArduinoJson.h>

/* Projeto Curto Circuito - ESP32: Sensor de Chuva */

#define SERVO_PIN 13
#define WIFI_SSID "motorola edge plus_8934"
#define WIFI_PASSWORD "123456789"
#define FIREBASE_AUTH "KCls06IBZBhXiOOxKRnVb738Xbgo8ZYH0BauSkLu"
#define FIREBASE_HOST "hhttps://automatic-window-cf750-default-rtdb.firebaseio.com/"

Servo servoMotor;

/* Sensor de Chuva - Pinagem e variáveis */
int pino_d = 12; /* Pino ligado ao D0 do sensor */
int pino_a = 36; /* Pino ligado ao A0 do sensor */
int val_d = 0;   /* Armazena o valor lido do pino digital */
int val_a = 0;   /* Armazena o valor lido do pino analógico */
/* LED */
int pin = 2;  /* Vermelho Pino D2 do ESP32 */
int pin2 = 4; /* Azul Pino D4 do ESP32 */

int rpt = 1;
void setup() {
  Serial.begin(9600);
  /* Sensores INPUT */
  pinMode(pino_d, INPUT);
  pinMode(pino_a, INPUT);
  /* LEDs OUTPUT */
  pinMode(pin, OUTPUT);
  pinMode(pin2, OUTPUT);

  servoMotor.attach(SERVO_PIN);
  servoMotor.write(90); // Posiciona o servo motor no centro (stop)

  //Connect to WIFI
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.println("Connecting");
  while(WiFi.status() != WL_CONNECTED){
    Serial.println(".");
    delay(500);
  }
  Serial.println();
  Serial.println("Connected:");
  Serial.println(WiFi.localIP());
  Firebase.begin(FIREBASE_AUTH, FIREBASE_HOST);
}
int n = 0;

void loop() {
  
  delay(5000); // Atraso de 5 segundos

   servoMotor.write(90); // Garante que o servo motor está parado no início do loop
  /* Armazena os valores de leitura */
  val_a = analogRead(pino_a);

  delay(1000); // Atraso de 5 segundos

  /* Se a leitura analógica for menor que 300 */
  if (val_a < 2500) {      /* Chuva intensa */
    digitalWrite(pin, 90);  /* Desliga */
    digitalWrite(pin2, 1); /* Liga */
    Serial.println("Chuva Intensa");
    Serial.println(val_a);
    if (rpt == 1) {
      for (int pos = 180; pos >= 90; pos -= 1) {
        servoMotor.write(pos);
        delay(15);  // waits 15ms to reach the position
        rpt = 2;
      }
    }
  }
  /* Se a leitura analógica for menor que 500 e maior que 300 */
  if (val_a <= 4000 && val_a >= 2500) { /* Chuva moderada */
    digitalWrite(pin, 1);               /* Liga */
    digitalWrite(pin2, 1);              /* Liga */
    Serial.println("Chuva Moderada ou Chuvisco");
    Serial.println(val_a);
  }
  /* Se a leitura analógica for maior que 500 */
 if (val_a > 4000) {      /* Sem previsão de Chuva */
  digitalWrite(pin, 1);  /* Liga */
  digitalWrite(pin2, 0); /* Desliga */
  Serial.println("Sem previsão de chuva");
  Serial.println(val_a);
  if (rpt == 2) {
    for (int pos = 90; pos >= 0; pos -= 1) { // Diminuir os graus aqui (pos -= 1)
      // in steps of 1 degree
      servoMotor.write(pos);
      delay(15);  // waits 15ms to reach the position
      rpt = 1;
      }
    }
  }
  servoMotor.write(90); // Garante que o servo motor está parado no final do loop
}
