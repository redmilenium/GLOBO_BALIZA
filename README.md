# GLOBO_BALIZA

Este proyecto es la versión Low Cost del DANTESAT, un nanosat que principalmente se dedicó durante su vidad util a emitir  frases de la "Divina Comedia" de Dante.

En mi caso, el elegido ha sido Miguel de Cervantes y nuestro "Quijote".

El montaje, sin placa solar, pero con batería Lipo pesa aproximadamente 30 g. 

Para elevarlo se necesitaran globos que puedan contener al menos 60 litros de Helio (o unir varios de ellos tipo "UP" de Disney).

Explicación: cada litro de Helio es capaz de elevar 1 g. de peso, pero ademas de el equipo de balizaje hay que hacer ascender el peso de los globos.

Si los globos son metalizados, pesan mas, pero son capaces de mantener durante mas tiempo el Helio (aproximadamente 7 dias).

Los globos de latex, son menos pesados, pero pierden Helio mas rapidamente. (aproximadamente 2 dias).

A mas poder de ascensión, mas veloz será esta y mas alto subira la baliza.

No obstante, esto esta en pruebas y lo mejor será ir añadiendo globos hasta que "a ojo de buen cubero" le de el visto bueno.

Tambien tengo comprobar la duración de la batería LIPO, ya que no va a disponer de placa solar que la cargue.

He utilizado un arduino nano, al que le le quitado el chip que controla el USB y el regular de tensión.

![image](https://user-images.githubusercontent.com/48222471/228915551-94f1ddf4-b4a0-430f-ab87-c8e708da4901.png)

Esto es debido a que quiero reducir el consumo el máximo posible (y a que no había recibido los atmel 328 DIP28).

El modulo LORA es un RA01, que enciendo mediante D4 cuando lo voy a utilizar y apago durante los tiempos muertos.

Tambien utilizo la función Sleep del arduino Nano para reducir los consumos y posiblemente tambien elimine los led´s.

Dado que he eliminado el chip USB del Nano, la carga del soft, se hace via ISP.

Esquema electrico:

![image](https://user-images.githubusercontent.com/48222471/228909935-afdc4040-1626-4235-8931-c99687ae3ff5.png)

Vista del montaje:

![image](https://user-images.githubusercontent.com/48222471/228908769-51fd6307-4d74-4b01-b87f-98e461586e4d.png)

Prueba de vida de que los mensajes son recibidos como test en las TinyGS:

![image](https://user-images.githubusercontent.com/48222471/228908670-437fca5c-1ef1-4457-991f-f8e27e7f1951.png)
![image](https://user-images.githubusercontent.com/48222471/228918226-2514e1cf-1ea5-44a5-ae04-cddd2e6dc6aa.png)


Una curiosidad, esta programado para si la baliza recibe algun paquete de datos, los reenvia en lugar de los mensajes preprogramados.

Las frecuencias a utilizar no deberán "molestar" a otros servicios o satelites...

Y este es el soft que corre en el Arduino Nano:

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

