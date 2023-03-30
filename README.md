# GLOBO_BALIZA

Este proyecto es la versión Low Cost del DANTESAT, un nanosat que principalmente se dedicó durante su vidad util a enviar frases de la "Divina Comedia" de Dante.

En mi caso, el elegido ha sido Miguel de Cervantes y nuestro "Quijote".

El montaje, sin placa solar, pero con batería Lipo pesa aproximadamente 30 g. 

Para elevarlo se necesitaran al menos globos que puedan contener al menos 60 litros de Helio (o unir varios de ellos tipo "UP" de Disney).

Explicación: cada litro de Helio es capaz de elevar 1 g. de peso, pero ademas de el equipo de balizaje hay que hacer ascender el peso de los globos.

Si los globos son metalizados, pesan mas, pero son capaces de mantener durante mas tiempo el Helio (aproximadamente 7 dias).

Los globos de latex, son menos pesados, pero pierden Helio mas rapidamente. (aproximadamente 2 dias).

A mas poder de ascensión, mas veloz será esta y mas alto subira la baliza.

No obstante, esto esta en pruebas y lo mejor será ir añadiendo globos hasta que "a ojo de buen cubero" le de el visto bueno.

He utilizado un arduino nano, al que le le quitado el chip que controla el USB y el regular de ten

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


