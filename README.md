# Ventilador Inteligente 

# Controle de Ambiente com Arduino

Este é um código Arduino para controle de ambiente que utiliza sensores de temperatura, umidade e luz, bem como um controle remoto infravermelho para ajustes. O código controla um motor com base na temperatura ambiente e exibe informações em um display LCD.

## Inclusão de Bibliotecas

```cpp
#include <EEPROM.h>
#include <LiquidCrystal_I2C.h>
#include <DHT_U.h>
#include <DHT.h>
#include <IRremote.h>
```

## Definição de Pinos

```cpp
#define pinSen 8
#define DHTTYPE DHT11
#define sensorLuz 7
#define velmotor 3
#define m1a 4
#define m1b 5
```

## Objetos e Variáveis Globais

```cpp
LiquidCrystal_I2C lcd(0x27,16, 2);
DHT sensor(pinSen,DHTTYPE);

unsigned long previousMillis = 0;
const long intervalo = 10000;
unsigned long currentMillis = millis();
float temp = 30.0;
int endereco = 0;
```

## Configuração Inicial

```cpp
void setup() {
  lcd.begin();
  pinMode(sensorLuz, INPUT);
  pinMode(m1a, OUTPUT);
  sensor.begin();
  float h = sensor.readHumidity();
  float t = sensor.readTemperature();
}
```

## Loop Principal

```cpp
void loop() {
  int info = digitalRead(sensorLuz);
  
  if(info == 1){
    lcd.backlight(); 
  } else{
    lcd.noBacklight(); 
  }
  
  if (IrReceiver.decode()) {
    int cod = IrReceiver.decodedIRData.command;
    
    switch(cod){
      case 22:
        setLCD();
        break;
      case 82:
        temp = temp - 10;
        setTemp();
        EEPROM.write(endereco, temp);
        setMotor(temp);
        break;
      case 24:
        temp = temp + 10;
        setTemp();
        EEPROM.write(endereco, temp);
        setMotor(temp);
        break;
      case 25:
        DesMotor();
        break;
      case 13:
        DesMotor();
        break;
    }
    
    IrReceiver.resume();
  }
}

// ... (continuação no próximo bloco)
```

## Funções de Controle

```cpp
void setLCD(){
  float h = sensor.readHumidity();
  float t = sensor.readTemperature();

  lcd.setCursor(0,0);
  lcd.print("TEMP");
  lcd.setCursor(5,0);
  lcd.print(t);
  lcd.setCursor(0,1);
  lcd.print("HUMI");
  lcd.setCursor(5,1);
  lcd.print(h);

  if (currentMillis - previousMillis >= intervalo){
    previousMillis = currentMillis;
    lcd.clear();
    delay(700);
  }
}

void setTemp(){
  lcd.setCursor(0,0);
  lcd.print("NOV TEMP");
  lcd.setCursor(9,0);
  lcd.print(temp);
}

void setMotor(float tmp){
  if(sensor.readTemperature() >= tmp){
    digitalWrite(m1a, HIGH);
  } else{
    digitalWrite(m1a, LOW);
  }
}

void DesMotor(){
  static bool est = true;
  
  if(est == true){
    digitalWrite(m1a, LOW);
    est = !est;
  } else{
    digitalWrite(m1a, HIGH);
    est = !est;
  }
}
```

