# Sistema_antincendios

![Arduino_Logo_Registered svg](https://user-images.githubusercontent.com/109388659/234407445-1de9faf7-fd9b-4d31-9f8d-089b83dd0892.png)
## Alumno

- Ivan Gonzalez de 1J

## Parcial practico de SPD
![image](https://github.com/Ivan-Ex-Gonzalez/Sistema_antincendios/assets/109388659/937b3c19-e656-4e71-a1d4-4c5e326edf46)


## Descripcion

- El objetivo de este proyecto es simular una  sistema de incendio utilizando Arduino que pueda detectar cambios de temperatura y activar un servo motor en caso de detectar un incendio.
- Además, se mostrará la temperatura actual y la estación del año  en un display LCD.

## Entradas digitales
![image](https://github.com/Ivan-Ex-Gonzalez/Sistema_antincendios/assets/109388659/256fdc17-804d-48d1-8200-98707608f065)

![image](https://github.com/Ivan-Ex-Gonzalez/Sistema_antincendios/assets/109388659/5508be43-4bc4-40d9-94ff-15a2a546b169)

![image](https://github.com/Ivan-Ex-Gonzalez/Sistema_antincendios/assets/109388659/524ec16f-41f5-4281-9f17-e655853cece7)



- Control remoto IR
- Sensor de temperatura TMP36

## Salidas digitales

![image](https://github.com/Ivan-Ex-Gonzalez/Sistema_antincendios/assets/109388659/80569b94-2936-46f8-b8ba-e9cdeb1f1056)
![image](https://github.com/Ivan-Ex-Gonzalez/Sistema_antincendios/assets/109388659/2ec1dca6-efaf-4704-b11b-fe519874de5b)



- 1 Led con una resistencia de 220 ohms en cada uno.

- 1 Led rgb con 3 resistencias de 220 ohms.

- 1 Servomotor

## Codigo

- Bibliotecas
```c++
  #include <Servo.h>
  #include <IRremote.h>
  #include <LiquidCrystal_I2C.h>
```
  
- Todas las definiciones al principio del codigo:
```c++
#define led_r 6
#define led_g 3
#define led_b 5
#define led_o 2
#define pin_servo 9
#define sensor_temp A0
#define pin_control 11
#define power 4278238976
```
- Todas las variables globales

```c++
int estado = 1;
int angulo_servo = 0;
bool boton = true;
int led_encendido = led_g;
int led_encendido_a = led_encendido;
int estacion_actual;
int temperatura_vieja = 1;
```
- Utilizo LiquidCrystal_I2C lcd(32,16,2); para determinar la direccion y el alcance de mi LCD

- La funcion alerta_servo()) se encarga de hacer girar el servomotor de  180 grados continuamente gracias a la variable angulo_servo.
```c++
void alerta_servo(){
  int giro = angulo_servo % 180;
  servo.write(giro);
  angulo_servo += 5;
}
```
- La funcion cartel_alerta() se encarga de mostrar en el LCD una alerta de incendio.
```c++
void cartel_alerta(){
  lcd.setCursor(0,0);
  lcd.print("     ALERTA     ");
  lcd.setCursor(0,1);
  lcd.print(" FUEGO DETECTADO");
}
```
- La funcion cartel_normal() recibe por parametro la temperatura detectada por el sensor , lo muestra en pantalla y llama a la funcion imprimirEstacion().
```c++
void cartel_normal(int temperatura){
	lcd.setCursor(7,0);
  	lcd.print(temperatura);
  	imprimirEstacion();
}
```
- La funcion borrador()) se encarga de borrar la primera linea del LCD.
```c++
void borrador(){
        lcd.setCursor(0,0);
  		lcd.print("                ");
}
```
- La funcion control_IR() se encarga de detectar si se presiona el boton power para invertir el balor de 'boton'.
```c++
void control_IR(){
  if (IrReceiver.decode()) {
    if (IrReceiver.decodedIRData.decodedRawData == power){
      boton = !boton;
      }
		IrReceiver.resume();
	}
		delay (100);
	}
```
- La funcion control_led_rgb() recibe temperatura y se encarga modificar cual es el color que debe de estar prendido actualmente, apagando el anterior'.
```c++
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
```
- La funcion led_aleta() se encarga de detectar se preciono el boton power para asi avisar si el sistema de alarma esta encendio o apagado.
```c++
void led_aleta(){
    if(boton){
    	digitalWrite(led_o, HIGH);
	}
	else{
		digitalWrite(led_o, LOW);
	}
}
```

- La funcion estacion() recibe temperatura y se encarga de darle un valor a la variable estacion_actual dependiendo de la temperatura actual.
```c++
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
```

- La funcion imprimirEstacion() se encarga imprimir en la linea 2 del LCD la estacio dependiendo de la temperatura.
```c++
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
```

- La funcion sistema_anti_incendios() recibe temperatura y se encarga de determinar si se muestra el cartel de alerta o el cartel normal.
```c++
void sistema_anti_incendios(int temperatura){
if (temperatura > 60 && boton){
  cartel_alerta();
  alerta_servo();      
  estado = 2;
  }
  else{
    if (temperatura_vieja != temperatura || estado == 2){
    	borrador();
    }
  	cartel_normal(temperatura);
  	estado = 1;
    temperatura_vieja = temperatura;
  }
}
```

## :eight_pointed_black_star:Link al proyecto

[Proyecto] https://www.tinkercad.com/things/7Het7feB37j
