# GLOBO_BALIZA

Este proyecto es la versión Low Cost del DANTESAT, un nanosat que principalmente se dedicó durante su vidad util a emitir  frases de la "Divina Comedia" de Dante.

info en https://tinygs.com/satellite/DANTEsat

En mi caso, el elegido ha sido Miguel de Cervantes y nuestro "Quijote".

El montaje, sin placa solar, pero con batería Lipo pesa aproximadamente 30 g. 

Para elevarlo se necesitaran globos que puedan contener al menos 60 litros de Helio (o unir varios de ellos tipo "UP" de Disney).

Explicación: cada litro de Helio es capaz de elevar 1 g. de peso, pero ademas de el equipo de balizaje hay que hacer ascender el peso de los globos.

Si los globos son metalizados, pesan mas, pero son capaces de mantener durante mas tiempo el Helio (aproximadamente 7 dias).

Los globos de latex, son menos pesados, pero pierden Helio mas rapidamente. (aproximadamente 2 dias).

A mas poder de ascensión, mas veloz será esta y mas alto subira la baliza.

No obstante, esto esta en pruebas y lo mejor será ir añadiendo globos hasta que "a ojo de buen cubero" le de el visto bueno.

Tambien tengo comprobar la duración de la batería LIPO, ya que no va a disponer de placa solar que la cargue.

Para limitar el peso, en el caso de utilizar una placa solar, tampoco he incluido ningun circuito que regule la carga de la bateria.

VERSION CON ARDUINO NANO

He utilizado un arduino nano, al que le le quitado el chip que controla el USB y el regular de tensión.

![image](https://user-images.githubusercontent.com/48222471/228915551-94f1ddf4-b4a0-430f-ab87-c8e708da4901.png)

Esto es debido a que quiero reducir el consumo el máximo posible (y a que no había recibido todavía los atmega 328U DIP28).

El modulo LORA es un RA01, que enciendo mediante D4 cuando lo voy a utilizar y apago durante los tiempos muertos.

Tambien utilizo la función Sleep del arduino Nano para reducir los consumos y posiblemente tambien elimine los led´s.

He utilizado una bateria de 3,7 voltios (cargada al máximo llega a 4,20 voltios) 500 mA de capacidad.

El modulo Lora se alimenta a traves de un diodo con una caida de 0,6 voltios, por tanto, a la máxima carga de la batería el módulo se alimentará a 3,6 voltios que segun especificaciones del fabricante es su limite superior.

El arduino nano esta configurado para que el sistema de BrownOut este en su limite inferior de tensión y no se reinicie aunque la batería este practimente descargada.

Esto se hace con la configuración de los fusibles e implica la necesidad de disponer de un programador HVPP

![image](https://user-images.githubusercontent.com/48222471/228961570-cb658775-fc18-4a5b-b02c-3d2527c8fe56.png)

Dado que he eliminado el chip USB del Nano, la carga del soft, se hace via ISP.

Esquema electrico:

![image](https://user-images.githubusercontent.com/48222471/228959022-5b7c2d62-fdad-40b6-97ce-a100f6fa1ecd.png)

Vista del montaje:

![image](https://user-images.githubusercontent.com/48222471/228908769-51fd6307-4d74-4b01-b87f-98e461586e4d.png)

Prueba de vida de que los mensajes son recibidos como test en las TinyGS:

![image](https://user-images.githubusercontent.com/48222471/228908670-437fca5c-1ef1-4457-991f-f8e27e7f1951.png)
![image](https://user-images.githubusercontent.com/48222471/228918226-2514e1cf-1ea5-44a5-ae04-cddd2e6dc6aa.png)

VERSION ATMEGA328P DIP28

He configurado mediante fusibles la utilización del reloj interno de 8 Mhz y asi evito un componente, el cuarzo.
![Imagen de WhatsApp 2023-04-01 a las 19 37 57](https://user-images.githubusercontent.com/48222471/229313613-2299445d-73c2-499b-97db-9801bfb96904.jpg)

El resto del circuito es practicamente como con el Arduino Nano.

Esquema electrico

![image](https://user-images.githubusercontent.com/48222471/230391784-cbda0b87-f617-421e-a1e1-c5893c1c92fb.png)

Vista General

![image](https://user-images.githubusercontent.com/48222471/229313185-77cf7366-6b5b-4c11-b76d-0601d74be83c.png)

Prueba de vida de que los mensajes son recibidos como test en las TinyGS:

![Imagen de WhatsApp 2023-04-01 a las 19 56 11](https://user-images.githubusercontent.com/48222471/229313222-dd1b40d4-3982-42db-a7fb-11f40d9111f4.jpg)
![Imagen de WhatsApp 2023-04-01 a las 19 56 52](https://user-images.githubusercontent.com/48222471/229313238-5e608e25-c7cb-445d-ab14-402e8d299eb2.jpg)



Una curiosidad, esta programado para si la baliza recibe algun paquete de datos, los reenvia en lugar de los mensajes preprogramados.

Las frecuencias a utilizar no deberán "molestar" a otros servicios o satelites...

Y este es el soft que corre en el Arduino Nano/Atmega 328P DIP28:

```

#include <Arduino.h>
#include <RadioLib.h>
#include <LowPower.h>

// SX1278                      ARDUINO NANO
// NSS   -------------------      D10
// DIO0  -------------------       D2
// RESET  ------------------       D9
// 
// CLK  --------------------      D13
// MISO --------------------      D12
// MOSI --------------------      D11

// OJO!! EL ARDUINO NANO FUNCIONA A 5 VOLTIOS Y EL MODULO LORA A 3.3
// LA CONEXIONES DE DATOS, LA HE PUESTOS DIRECTAS, PERO LA ALIMENTACION SI HAY QUE RESPETARLA
//POWER --------------------      D4  -> ACTIVA UN TRANSISTOR 
//GND -----------------------     GND

// CON EL FIN DE REDUCIR EL CONSUMO, HE ELIMINADO EL REGULADOR DE 5V DEL ARDUINO NANO Y EL CHIP DE USB
// POR TANTO, EL PROGRAMA DEBERÁ ESTAR CARGADO ANTES DE QUITAR EL CHIP USB O UTILIZAR UN PROGRAMADO ISP

SX1278 radio = new Module(10, 2, 9);

int state;
volatile bool receivedFlag, enableInterrupt;
unsigned long timeout,resetrx;
String txt="TinyGS-test ";  // 
const char frase0[] PROGMEM="En un lugar de la mancha, de cuyo nombre no quiero acordarme...";
const char frase1[] PROGMEM="¡Oh, memoria, enemiga mortal de mi descanso!";
const char frase2[] PROGMEM="La virtud más es perseguida de los malos que amada de los buenos";
const char frase3[] PROGMEM="Mire vuestra merced -respondio Sancho- que aquellos que alli se parecen no son gigantes, sino molinos de viento";
const char frase4[] PROGMEM="La ingratitud es hija de la soberbia";
const char frase5[] PROGMEM="La razon de la sinrazon que a mi razon se hace, de tal manera mi razon enflaquece, que con razon me quejo de la vuestra fermosura";
const char frase6[] PROGMEM="Come poco y cena menos, que la salud de todo el cuerpo se fragua en la oficina del estomago";
const char frase7[] PROGMEM="La sangre se hereda y la virtud se aquista; y la virtud vale por si sola lo que la sangre no vale";
const char frase8[] PROGMEM="Esta que llaman por ahi Fortuna es una mujer borracha y antojadiza, y sobre todo, ciega, y asi no ve lo que hace, ni sabe a quien derriba";
const char frase9[] PROGMEM="Las tristezas no se hicieron para las bestias, sino para los hombres; pero si los hombres las sienten demasiado, se vuelven bestias";
const char frase10[] PROGMEM="La pluma es lengua del alma; cuales fueren los conceptos que en ella se engendraron, tales serán sus escritos";
const char frase11[] PROGMEM="¡Venturoso aquel a quien el cielo dio un pedazo de pan, sin que le quede obligacion de agradecerselo a otro que al mismo cielo!";
const char frase12[] PROGMEM="Por la libertad, asi como por la honra, se puede y se debe aventurar la vida";
const char frase13[] PROGMEM="Confia en el tiempo, que suele dar dulces salidas a muchas amargas dificultades";
const char frase14[] PROGMEM="Bebo cuando tengo gana, y cuando no la tengo y cuando me lo dan, por no parecer o melindroso o malcriado";
const char frase15[] PROGMEM="Y asi, del poco dormir y del mucho leer, se le seco el cerebro";
const char frase16[] PROGMEM="El año que es abundante de poesia, suele serlo de hambre";
const char frase17[] PROGMEM="Si acaso doblares la vara de la justicia, no sea con el peso de la dadiva, sino con el de la misericordia";
const char frase18[] PROGMEM="Y vera el mundo que tiene contigo más fuerza la razon que el apetito";
const char frase19[] PROGMEM="Sabete, Sancho, que no es un hombre más que otro si no hace mas que otro";
const char frase20[] PROGMEM="No huye el que se retira";
const char frase21[] PROGMEM="Casamientos de parientes tienen mil inconvenientes";

const char* const tabla_textos[] PROGMEM = {frase0, frase1, frase2, frase3, frase4, frase5, frase6,frase7,frase8,frase9,frase10,frase11,frase12,frase13,frase14,frase15,frase16,frase17,frase18,frase19, frase20,frase21};
 
char buffer[300];  // 

//*********************************************************************************************************
void rx()
{
    // disable the interrupt service routine while
    // processing the data
    enableInterrupt = false;
    // reset flag
    receivedFlag = false;
     String str;
     int state = radio.readData(str);
     Serial.println(str);
     txt=str;

  if (state == RADIOLIB_ERR_NONE) 
  {
    // packet was successfully received
    Serial.println(F("success!"));

    // print the data of the packet
    Serial.print(F("[SX1278] Data:\t\t"));
    Serial.println(str);

    // print the RSSI (Received Signal Strength Indicator)
    // of the last received packet
    Serial.print(F("[SX1278] RSSI:\t\t"));
    Serial.print(radio.getRSSI());
    Serial.println(F(" dBm"));

    // print the SNR (Signal-to-Noise Ratio)
    // of the last received packet
    Serial.print(F("[SX1278] SNR:\t\t"));
    Serial.print(radio.getSNR());
    Serial.println(F(" dB"));

    Serial.print(F("[SX1278] LONGITUD:\t\t"));
    Serial.print(radio.getPacketLength());
    Serial.println(F(" bytes"));
    Serial.println();


  } else if (state == RADIOLIB_ERR_RX_TIMEOUT) {
    // timeout occurred while waiting for a packet
    Serial.println(F("timeout!"));
 

  } else if (state == RADIOLIB_ERR_CRC_MISMATCH) {
    // packet was received, but is malformed
    Serial.println(F("CRC error!"));
 

  } else {
    // some other error occurred
    Serial.print(F("failed, code "));
    Serial.println(state);
   }

    receivedFlag = false;
    enableInterrupt = true;
    radio.startReceive();
   
}

void tx()
{
    Serial.print(F("[SX1278] Transmitting packet ... "));
    state= radio.transmit(txt);
    
    if (state == RADIOLIB_ERR_NONE) 
    {
    // the packet was successfully transmitted
    Serial.println(F(" success!"));
    // print measured data rate
    Serial.print(F("[SX1278] Datarate:\t"));
    Serial.print(radio.getDataRate());
    Serial.println(F(" bps"));
    Serial.println();
    } 
     else 
    {
    // some other error occurred
    Serial.print(F("failed, code "));
    Serial.println(state);
    }
   
       receivedFlag = false;
       enableInterrupt = true;
       radio.startReceive();
}
//*****************************************************************************
  void setFlag() 
  {
  // check if the interrupt is enabled
  if(!enableInterrupt)
   {
    return;
   }
  // we got a packet, set the flag
    receivedFlag = true;  
  }
//*****************************************************************************
void iniciar()
{
  SPI.begin();
  Serial.println("  ");
  Serial.println("[SX1278] Initializing ... ");

  state = radio.begin(400.450,500,9,5,18,17,8); // GAOFEN
  //state = radio.begin(400.130,500,9,5,18,17,8); // GAOFEN19
  //state = radio.begin(433.090,62.5,8,5,18,17,8); // HAB_ISM_433,09_M2

 
 if (state == RADIOLIB_ERR_NONE) 
  {
    Serial.print(" -> HECHO!");
    Serial.println();
  } else {
    Serial.print("MECACHIS :failed, code ");
    Serial.println(state);
    //while (true);
  }
 
   radio.setDio0Action(setFlag);
   radio.forceLDRO(true);
   //radio.autoLDRO();
   radio.setCRC(true);
   radio.startReceive();
   
    // we're ready to receive more packets,
    // enable interrupt service routine
   
   receivedFlag = false;
   enableInterrupt = true;
   timeout=millis();
   resetrx=millis();
}

void setup() 
{
  //attachInterrupt(digitalPinToInterrupt(2), setFlag, FALLING);

  // cuando arranco espero dormido durante 8 segundos por 5 = 40 segundos
  // para dar tiempo a que se estabilice la tensión
  //for (size_t i = 0; i < 5; i++)
  //{
  //  LowPower.powerDown(SLEEP_8S, ADC_OFF, BOD_OFF); 
  // }
 
  Serial.begin(9600);
  // el pin D4 se utilizará para activar un transistor que encienda el modulo LORA
  pinMode(4,OUTPUT);

// CLK --------  D13
// MISO -------  D12
// MOSI -------  D11
// SS/NSS -----  D10

}
//*****************************************************************************

void loop() 
{
  //apago el modulo Lora
  digitalWrite(4,LOW);
  //duermo el procesador el tiempo que estime
  LowPower.longPowerDown(100000); // 100 segundos en este caso
  timeout=millis();
  //enciendo el modulo Lora
  digitalWrite(4,HIGH);
  //le doy tiempo para que se despereze
  delay(500);
  
  //configuro el modulo LORA
  iniciar();

  int i= random(0,22);// selecciono una frase del Quijote al "azar"

  strcpy_P(buffer, (char*)pgm_read_word(&(tabla_textos[i])));//i
  String Quijote=String(buffer);

  //precargo el mensaje a enviar
  txt="TinyGS-test "; // esta linea esta incluida para se gestione el paquete como TEST
  txt+=Quijote;
  //Serial.println(txt);
  //si recibo algo, lo guardo como mensaje a enviar
  while(millis()-timeout<60000)
  {
    if(receivedFlag)  rx(); // si recibo algo, paso a utilizarlo como dato a enviar
  }
  //envio o el mensaje precargado o lo que haya recibido
  //y aqui si hay libertad de expresión: lo que reciba será retransmitido
  tx(); 
  //espero un poco 
  delay(2000);
   // vuelvo a repetir el bucle
}
//*****************************************************************************

```
RESULTADO PRUEBAS EN TIERRA
Con una LIPO de 500 mA. que en realidad carga 469 mA, he obtenido una duración de aproximadamente 30 horas y se han enviado unos 450 Quijotescos mensajes.
Es decir, por cada miliamperio real de carga de la LIPO, se ha podido enviado un mensaje.
O tambien que se necesitan 150 mA por cada 10 horas de operatividad y en ese tiempo se enviaran 150 mensajes.
A mayor capacidad de la batería, mayor autonomía, pero tambien mayor peso y la necesidad de utilizar mas Helio para poder elevarlo todo.

