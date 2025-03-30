# 🚧 Projeto: Cancela Automática com Sensor Ultrassônico + Botão

Este projeto implementa um sistema de **cancela automática** controlado por **Arduino UNO**, que só abre quando:
1. **Uma presença for detectada** a menos de 40 cm pelo sensor ultrassônico HC-SR04, **e**
2. **O botão físico for pressionado** simultaneamente.

> ✅ Maior controle de acesso  
> 🔒 Segurança contra ativações indesejadas  
> 📟 Feedback visual via LCD

---

## 🔌 Componentes Utilizados

- Arduino UNO R3
- Sensor Ultrassônico HC-SR04
- Servo motor SG90
- Push Button
- Display LCD 16x2 com módulo I2C
- Protoboard + jumpers

---

## 🔧 Ligações dos Componentes

| Componente      | Pino Arduino |
|------------------|--------------|
| HC-SR04 TRIG     | 12           |
| HC-SR04 ECHO     | 11           |
| Botão (Pull-up)  | 7            |
| Servo SINAL      | 9            |
| LCD SDA          | A4           |
| LCD SCL          | A5           |

> O botão está ligado entre **pino 7** e **GND**, com `INPUT_PULLUP` ativado no código.

---

## 💻 Lógica do Código

```cpp
#include <Servo.h>
#include <LiquidCrystal_I2C.h>

Servo myServo;
LiquidCrystal_I2C lcd(0x27, 16, 2);

const int buttonPin = 7;
const int trigPin = 12;
const int echoPin = 11;
const int servoPin = 9;

bool isOpen = false;

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buttonPin, INPUT_PULLUP);  // Botão com resistor interno

  myServo.attach(servoPin);
  myServo.write(90);  // Inicia com cancela fechada

  Serial.begin(9600);
  Wire.begin();
  lcd.begin(16, 2);
  lcd.backlight();
  lcd.clear();
  lcd.print("Sistema pronto");
  delay(2000);
  lcd.clear();
}

long medirDistancia() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH);
  return duration * 0.034 / 2;
}

void loop() {
  long distancia = medirDistancia();
  bool estadoBotao = digitalRead(buttonPin) == LOW;
  Serial.print("Distancia: ");
  Serial.print(distancia);
  Serial.print(" cm | Botao: ");
  Serial.println(estadoBotao ? "PRESSIONADO" : "NAO");

  if (!isOpen) {
    if (distancia < 40) {
      lcd.clear();
      lcd.print("Pessoa detectada");
      Serial.println("Presenca detectada");
      delay(1000);

      lcd.clear();
      lcd.print("Aperte o botao");
      Serial.println("Esperando botao");

      unsigned long tempoInicio = millis();
      bool botaoPressionado = false;

      // Espera até 10 segundos o botão ser pressionado, desde que a pessoa ainda esteja lá
      while (millis() - tempoInicio < 10000) {
        distancia = medirDistancia();
        estadoBotao = digitalRead(buttonPin) == LOW;

        if (distancia >= 40) {
          lcd.clear();
          lcd.print("Pessoa saiu");
          Serial.println("Pessoa saiu");
          delay(1000);
          return;  // Sai da função loop e volta ao início
        }

        if (estadoBotao) {
          botaoPressionado = true;
          break;
        }

        delay(1000);
      }

      if (botaoPressionado) {
        lcd.clear();
        lcd.print("Abrindo cancela");
        Serial.println("Abrindo cancela");
        myServo.write(0);
        isOpen = true;
        delay(3000);
      } else {
        lcd.clear();
        lcd.print("Tempo esgotado");
        Serial.println("Botao nao pressionado");
        delay(2000);
      }

    } else {
      lcd.clear();
      lcd.print("Aguardando pessoa");
      delay(500);
    }
  }

  // FECHAR cancela se já estiver aberta e a pessoa foi embora
  if (isOpen) {
    distancia = medirDistancia();
    if (distancia >= 40) {
      lcd.clear();
      lcd.print("Fechando cancela");
      Serial.println("Fechando cancela");
      myServo.write(90);
      isOpen = false;
      delay(2000);
    } else {
      lcd.clear();
      lcd.print("Acesso liberado");
    }
  }

  delay(300);
}

}
