# esp32-IRreceiverRAW-PTZKamera
esp32-IRreceiverRAW für PTZ Kamera HDKatov
```cpp
#include <IRremote.hpp>

#define IR_SEND_PIN 4  // GPIO für Sende-LED (z. B. KY-005)

unsigned long stateStartTime = 0;
int state = 0;

// === Exakte RAW-Daten von der Fernbedienung ===

// AI Toggle (0x6A49)
uint16_t raw_ai_toggle[] = {
  2350,650, 1150,700, 500,650, 550,700,
  1100,700, 500,750, 450,700, 1150,650,
  550,700, 500,700, 1150,650, 550,650,
  1150,650, 550,650, 1150,700, 1100
};

// PTZ Rechts (0x51D1C)
uint16_t raw_ptz_right[] = {
  2300,700, 500,700, 500,700, 1150,650,
  1150,650, 1150,650, 550,650, 500,750,
  450,750, 1100,700, 550,650, 1150,650,
  1150,650, 1150,700, 500,700, 500,700,
  500,700, 1050,750, 550,650, 1100,700, 550
};

// PTZ Links (0x51D1D)
uint16_t raw_ptz_left[] = {
  2400,600, 1200,650, 550,600, 1200,650,
  1200,600, 1150,650, 550,650, 600,600,
  600,600, 1200,650, 550,650, 1150,600,
  1200,650, 1150,650, 550,650, 550,650,
  550,700, 1150,650, 550,650, 1150,650, 550
};

void setup() {
  Serial.begin(115200);
  delay(200);
  IrSender.begin(IR_SEND_PIN, ENABLE_LED_FEEDBACK, USE_DEFAULT_FEEDBACK_LED_PIN);
  Serial.println("🔧 Starte IR-Sendesequenz mit echten RAW-Codes.");
}

void loop() {
  unsigned long now = millis();

  switch (state) {
    case 0:  // AI TOGGLE senden
      Serial.println("🟣 Sende: AI TOGGLE (3×)");
      sendRawMultiple(raw_ai_toggle, sizeof(raw_ai_toggle) / sizeof(raw_ai_toggle[0]), 3);
      stateStartTime = now;
      state = 1;
      break;

    case 1:
      if (now - stateStartTime >= 10000) {
        state = 2;
      }
      break;

    case 2:  // PTZ RECHTS senden
      Serial.println("🟦 Sende: PTZ RECHTS (10×)");
      sendRawMultiple(raw_ptz_right, sizeof(raw_ptz_right) / sizeof(raw_ptz_right[0]), 10);
      stateStartTime = now;
      state = 3;
      break;

    case 3:
      if (now - stateStartTime >= 10000) {
        state = 4;
      }
      break;

    case 4:  // PTZ LINKS senden
      Serial.println("🟥 Sende: PTZ LINKS (10×)");
      sendRawMultiple(raw_ptz_left, sizeof(raw_ptz_left) / sizeof(raw_ptz_left[0]), 10);
      stateStartTime = now;
      state = 5;
      break;

    case 5:
      if (now - stateStartTime >= 10000) {
        Serial.println("🔁 Zyklus abgeschlossen – beginne von vorn...\n");
        state = 0;
      }
      break;
  }
}

// Hilfsfunktion für wiederholtes Senden
void sendRawMultiple(uint16_t *rawData, size_t length, int times) {
  for (int i = 0; i < times; i++) {
    IrSender.sendRaw(rawData, length, 38); // 38 kHz Träger
    delay(100);
  }
}
```
Ausgabe:
```cpp
19:45:05.395 -> 
19:45:05.395 -> 🟣 Sende: AI TOGGLE (3×)
19:45:15.375 -> 🟦 Sende: PTZ RECHTS (10×)
19:45:25.374 -> 🟥 Sende: PTZ LINKS (10×)
19:45:35.381 -> 🔁 Zyklus abgeschlossen – beginne von vorn...
19:45:35.381 -> 
19:45:35.381 -> 🟣 Sende: AI TOGGLE (3×)
19:45:45.384 -> 🟦 Sende: PTZ RECHTS (10×)
19:45:55.375 -> 🟥 Sende: PTZ LINKS (10×)
19:46:05.385 -> 🔁 Zyklus abgeschlossen – beginne von vorn...
```
