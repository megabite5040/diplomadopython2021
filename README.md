# diplomadopython2021

from machine import Pin, I2C, ADC
from ssd1306 import SSD1306_I2C
from dht import DHT11
from utime import sleep
import network, time, urequests

sensorG = ADC(Pin(35))    #SENSOR DE GAS
Alarma= Pin(4, Pin.OUT)    #ALARMA EN EL PIN 4
led_rojo = Pin(2, Pin.OUT)     #LED ROJO EN EL PIN 2
led_amarillo = Pin(16, Pin.OUT)    #LED AZUL EN EL PIN 16
file = open("test.txt", "w")
sensorG.width(ADC.WIDTH_10BIT) #SENSOR 10 BIT
sensorG.atten(ADC.ATTN_11DB)   #SENSOR 11 DB

ancho = 128     #MEDIDA ANCHO PANTALLA OLED
alto = 64       #MEDIDA ALTO PANTALLA OLED

i2c = I2C(0, scl=Pin(22), sda=Pin(21))    #ASIGNACIÓN DE PIN A SENSOR I2C GAS
oled = SSD1306_I2C(ancho, alto, i2c)      #ENLACE DE PANTALLA CON LIBRERIA OLED

print(i2c.scan())   #IMPRIMIR 
 
########################################################################### 
oled.text(' - BIENVENIDO -', 0, 0)  #BIENVENIDA DE SISTEMA
oled.text('Sistema de', 18, 20) #BIENVENIDA DE SISTEMA
oled.text('Control', 24, 30)    #BIENVENIDA DE SISTEMA
oled.text('de GAS', 28, 40)     #BIENVENIDA DE SISTEMA
oled.show()                    #DESPUES
sleep(4)                       #APAGAR

###########################################################################
def conectaWifi(red, password):     #CONEXION A WIFI (RED Y EL PASSWORD)
     global miRed
     miRed = network.WLAN(network.STA_IF)     
     if not miRed.isconnected():              #Si no está conectado…
          miRed.active(True)                   #activa la interface
          miRed.connect('familiaruiz', 'Tatan57680')         #Intenta conectar con la red
          print('Conectando a la red', red +"…")
          timeout = time.time ()
          while not miRed.isconnected():           #Mientras no se conecte..
              if (time.ticks_diff (time.time (), timeout) > 10):
                  print ('no se pudo conectar a la red')
                  return False
     return True
############################################################################
i2c = I2C(0, scl=Pin(22), sda=Pin(21))    #ASIGNACIÓN DE PIN A SENSOR I2C GAS

if conectaWifi("familiaruiz", "Tatan57680"):
    
     print("Conexión exitosa!")
     print('Datos de la red (IP/netmask/gw/DNS):', miRed.ifconfig())
    
     url = "https://maker.ifttt.com/trigger/sensor_gas/with/key/b_KmzTm2wJGSVnoCiobiMa?" 

############################################################################# 
while True:
    
 sensorG_value = float(sensorG.read())
 oled.fill(0)
 led_amarillo.value (1)
 oled.text('   SENSOR DE GAS', 0, 0)
 oled.text(' NIVEL:'+ str(sensorG_value), 0,20)
 print ('SENSOR DE GAS')
 print('NIVEL:'+ str(sensorG_value))
 print(sensorG_value > 300)
 sleep(0.01)
 respuesta = urequests.get(url+"&value1="+str(sensorG_value))  #ENVIO DE DATOS A TABLA DRIVE
 print(respuesta.text)                                         #ENVIO DE DATOS A TABLA DRIVE
 print (respuesta.status_code)                                 #ENVIO DE DATOS A TABLA DRIVE
 respuesta.close ()
 ##############################################################################
 if sensorG_value > 300:
     
     
     oled.text(' ALERTA!', 0,40)
     oled.text(' FUGA DE GAS!', 0,50)
     led_amarillo.value (0)
     led_rojo.value (1)
     Alarma.on()
     time.sleep_ms(300)
     led_amarillo.value (1)
     led_rojo.value (0)
     oled.show()
     time.sleep (0.25)
     ##### GMAIL 
     
 oled.show()
 sleep(0.25)
 Alarma.off()
 time.sleep(0.25)
 
else:
    print ("Imposible conectar")
    miRed.active (False)
 
