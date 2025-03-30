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
if (presencaDetectada && botaoPressionado && !isOpen) {
    // Abrir a cancela
}
