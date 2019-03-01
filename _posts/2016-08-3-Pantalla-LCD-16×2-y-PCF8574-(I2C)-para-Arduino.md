---
layout: post
title: "Pantalla LCD 16×2 y PCF8574 (I2C) para Arduino"
date: 2016-08-2 16:25:06 -0700
comments: true
---

En esta entrada tratare de una forma breve el uso de una pantalla LCD 16x2 con PCF8574 por i2c.

Lo primeo sera identificar la dirección de nuestra pantalla utilizando el siguiente código:
```
    #include <Wire.h>
     
     
    void setup()
    {
      Wire.begin();
     
      Serial.begin(9600);
      while (!Serial);             // Leonardo: wait for serial monitor
      Serial.println("\nI2C Scanner");
    }
     
     
    void loop()
    {
      byte error, address;
      int nDevices;
     
      Serial.println("Scanning...");
     
      nDevices = 0;
      for(address = 1; address < 127; address++ )
      {
        // The i2c_scanner uses the return value of
        // the Write.endTransmisstion to see if
        // a device did acknowledge to the address.
        Wire.beginTransmission(address);
        error = Wire.endTransmission();
     
        if (error == 0)
        {
          Serial.print("I2C device found at address 0x");
          if (address<16)
            Serial.print("0");
          Serial.print(address,HEX);
          Serial.println("  !");
     
          nDevices++;
        }
        else if (error==4)
        {
          Serial.print("Unknow error at address 0x");
          if (address<16)
            Serial.print("0");
          Serial.println(address,HEX);
        }    
      }
      if (nDevices == 0)
        Serial.println("No I2C devices found\n");
      else
        Serial.println("done\n");
     
      delay(5000);           // wait 5 seconds for next scan
    }
```
Conectaremos las salidas SLC, SDA, GND a negativo y VCC la conectaremos a una de las salidas de 5V.

Una vez compilado y subido el código anterior, podremos abrir nuestro monitor serial que se encuentra el el apartado herramientas del IDE de Arduino.

Nos mostrara algo así:
```
Scanning...
I2C device found at address 0x3F  !
done
```

Con esto tendremos la dirección de nuestra pantalla, ya solo quedaría probarla con un código de ejemplo, recuerda tener instalada la librería LiquidCrystal.
```
#include <Wire.h>                 
#include <LiquidCrystal_I2C.h>   
LiquidCrystal_I2C lcd(0x3F, 2, 1, 0, 4, 5, 6, 7, 3, POSITIVE);//Direccion de LCD
void setup()   { 
lcd.begin(16,2);// Indicamos medidas de LCD        
}
void loop() {
lcd.clear();//Elimina todos los simbolos del LCD
lcd.setCursor(0,0);//Posiciona la primera letra despues del segmento 5 en linea 1             
lcd.print("AventuraBinaria");
delay (2000);//Dura 2 segundos
lcd.clear();
lcd.setCursor(3,1);//Posiciona la primera letra despues del segmento 6 en linea 2            
lcd.print("Hola Mundo!");
delay (2000);//Dura 1 segundo  
}
```

Un saludo!
