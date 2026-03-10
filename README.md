
#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>

RF24 radio(9, 10); // CE, CSN
const byte address[6] = "00001";

struct TelemetryPacket {
  uint16_t xValue;
  uint16_t yValue;
  unsigned long timestamp;
};

// Піни для L298N
int IN1 = 4;
int IN2 = 5;
int IN3 = 6;
int IN4 = 7;

void setup() {
  Serial.begin(115200);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  if (!radio.begin()) {
    Serial.println("NRF24L01 error");
    while (true);
  }

  radio.setPALevel(RF24_PA_HIGH);
  radio.setDataRate(RF24_250KBPS);
  radio.setChannel(120);
  radio.openReadingPipe(0, address);
  radio.startListening();

  Serial.println("Приймач готовий, слухаю...");
}

void loop() {
  if (radio.available()) {
    TelemetryPacket packet;
    radio.read(&packet, sizeof(packet));

    Serial.print("X="); Serial.print(packet.xValue);
    Serial.print(" Y="); Serial.println(packet.yValue);

    // Логіка руху через джойстик
    if (packet.xValue > 3000) { 
      digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
      digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
    } else if (packet.xValue < 1000) { 
      digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
      digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
    } else if (packet.yValue > 3000) { 
      digitalWrite(IN1, HIGH); digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW); digitalWrite(IN4, HIGH);
    } else if (packet.yValue < 1000) { 
      digitalWrite(IN1, LOW); digitalWrite(IN2, HIGH);
      digitalWrite(IN3, HIGH); digitalWrite(IN4, LOW);
    } else { // стоп
      digitalWrite(IN1, LOW); digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW); digitalWrite(IN4, LOW);
    }
  }
}
