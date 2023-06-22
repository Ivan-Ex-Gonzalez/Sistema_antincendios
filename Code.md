#CODIGO

```c++
// C++ code
//Ivan Gonzalez 1
#include <Servo.h>
#include <IRremote.h>
#include <LiquidCrystal_I2C.h>

#define led_r 6
#define led_g 3
#define led_b 5
#define led_o 2
#define pin_servo 9
#define sensor_temp A0
#define pin_control 11
#define power 4278238976


int estado = 1;
int angulo_servo = 0;
bool boton = true;
int led_encendido = led_g;
int led_encendido_a = led_encendido;
int estacion_actual;
int temperatura_vieja = 1;

LiquidCrystal_I2C lcd(32,16,2);
Servo servo;
IRrecv irrecv(pin_control);
decode_results results;

void setup()
{
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  servo.attach(pin_servo);
  servo.write(0);
  IrReceiver.begin(pin_control, DISABLE_LED_FEEDBACK);
  pinMode(led_o, OUTPUT);
}

void loop()
{
  int lecturaSensor = analogRead(sensor_temp);
  int temperatura = map(lecturaSensor,20,358,-40,125);
 
  estacion(temperatura);
  
  control_led_rgb(temperatura);
  
  led_aleta();
  control_IR();
  
  sistema_anti_incendios(temperatura);
  
}


void alerta_servo(){
  int giro = angulo_servo % 180;
  servo.write(giro);
  angulo_servo += 5;
}

void cartel_alerta(){
  lcd.setCursor(0,0);
  lcd.print("     ALERTA     ");
  lcd.setCursor(0,1);
  lcd.print(" FUEGO DETECTADO");
}

void cartel_normal(int temperatura){
	lcd.setCursor(7,0);
  	lcd.print(temperatura);
  	imprimirEstacion();
}

void borrador(){
        lcd.setCursor(0,0);
  		lcd.print("                ");
}

void control_IR(){
  if (IrReceiver.decode()) {
    if (IrReceiver.decodedIRData.decodedRawData == power){
      boton = !boton;
      }
		IrReceiver.resume();
	}
		delay (100);
	}

void control_led_rgb(int temperatura){
  analogWrite(led_encendido, 255);
  if (temperatura < 0){
      led_encendido = led_b;
    }
  else if (temperatura > 0 && temperatura < 30){
      led_encendido = led_g;
    }
  else{
      led_encendido = led_r;
    }
  if (led_encendido_a != led_encendido){
  	analogWrite(led_encendido_a, 0);
  	led_encendido_a = led_encendido;
  }
 }

void led_aleta(){
    if(boton){
    	digitalWrite(led_o, HIGH);
	}
	else{
		digitalWrite(led_o, LOW);
	}
}

void estacion(int temp){
  if(temp < 1){
  estacion_actual = 1;
  } 
  else if (temp < 42){
  estacion_actual = 2;
  }
  else if (temp < 83){
  estacion_actual = 3;
  }
  else{
  estacion_actual = 4;
  }
}

void imprimirEstacion() {
  lcd.setCursor(0, 1);
  
  switch (estacion_actual) {
    case 1:
      lcd.print("    Invierno    ");
      break;
    case 2:
      lcd.print("     Otonio     ");
      break;
    case 3:
      lcd.print("    Primavera   ");
      break;
    case 4:
      lcd.print("     Verano     ");
      break;
    default:
      lcd.print("   Desconocido");
      break;
  }
}


void sistema_anti_incendios(int temperatura){
if (temperatura > 60 && boton){
  cartel_alerta();
  alerta_servo();      
  estado = 2;
  }
  else{
    if (temperatura_vieja != temperatura){
    	borrador();
    }
  	cartel_normal(temperatura);
  	estado = 1;
    temperatura_vieja = temperatura;
  }
}
```
