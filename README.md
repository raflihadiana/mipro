# Tugas Besar Mikroposesor IOT

## Task Information

**Judul**: Sistem Sensor Pendeteksi Gerak Dalam Implementasi Stabilizer Kamera

**Kelompok 8**: Rafli, Ikhwan, Wilda

### Aristektur Sistem 

![Parts](https://hackmd.io/_uploads/r1TqB2xVa.jpg)



### Cicuit Diagram

![image](https://hackmd.io/_uploads/S1lGBneET.png)


### Flowchart Program

![Flowchart Program.drawio](https://hackmd.io/_uploads/HyuwB3lEp.png)


### Program Board


1. Import Library Terkait
    ```c
    #include "LIS3DHTR.h"
    #include <Arduino.h>
    #include <U8x8lib.h>
    #ifdef SOFTWAREWIRE
        #include <SoftwareWire.h>
        SoftwareWire myWire(3, 2);
        LIS3DHTR<SoftwareWire> LIS;       //Software I2C
        #define WIRE myWire
    #else
        #include <Wire.h>
        LIS3DHTR<TwoWire> LIS;           //Hardware I2C
        #define WIRE Wire
    #endif
    ```

2. Inisiasi Variable Terkait
    ```c
    U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);

    const int potPin = A0; // Terserah Anda pada pin mana potensiometer dihubungkan
    const int ledPin = 4; // Terserah Anda pada pin mana LED dihubungkan
    const int BuzzerPin = 5; // Pin Buzzer
    ```

3. Inisiasi PinMode
```c
void setup() {
  Serial.begin(9600);
  pinMode(ledPin, OUTPUT);
  pinMode(BuzzerPin, OUTPUT);
  u8x8.begin();
  u8x8.setFlipMode(1);
  while (!Serial) {};
    LIS.begin(WIRE, 0x19); //IIC init
    delay(100);
    LIS.setOutputDataRate(LIS3DHTR_DATARATE_50HZ);
}
```

4. Logika Aplikasi
```c
void loop() {
  int potValue = analogRead(potPin); // Membaca nilai dari potensiometer (0 - 1023)

  // Mengubah nilai potensiometer menjadi rentang 50ms - 1000ms (0.05s - 1s)
  int delayTime = map(potValue, 0, 1023, 40, 1000);

  if (!LIS) {
        Serial.println("LIS3DHTR didn't connect.");
        while (1);
        return;
  }

  int x = LIS.getAccelerationX();
  int y = LIS.getAccelerationY();
  int z = LIS.getAccelerationZ();

  u8x8.setFont(u8x8_font_chroma48medium8_r);
  u8x8.setCursor(0, 0);
  
  if (x<=0.05 && x>=0 && y>=-0.05 && y<=0 && z<=1.05 && z>=1) {
    digitalWrite(ledPin, LOW); // Mengaktifkan LED
    analogWrite(BuzzerPin, 0);
    u8x8.print("SISTEM STABIL");
    delay(delayTime); // Menunggu sesuai delay
  }
  else {
    digitalWrite(ledPin, HIGH); // Mematikan LED
    analogWrite(BuzzerPin, 128);
    u8x8.print("TIDAK STABIL");
    delay(delayTime); // Menunggu sesuai delay lagi
  }

  Serial.print("x:"); Serial.print(LIS.getAccelerationX()); Serial.print("  ");
  Serial.print("y:"); Serial.print(LIS.getAccelerationY()); Serial.print("  ");
  Serial.print("z:"); Serial.println(LIS.getAccelerationZ());

}
```
