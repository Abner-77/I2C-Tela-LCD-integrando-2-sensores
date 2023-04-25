# I2C-Tela-LCD-integrando-2-sensores
Projeto de aula. Prototipo com modelagem em simulador , integração protocolo I2C com tela LCD e sensores termistor NTC e LDR .

from machine import SoftI2C, Pin, ADC
from mp_i2c_lcd1602 import LCD1602
from time import sleep_ms
from math import log


# Inicialize o I2C e o display LCD1602
i2c = SoftI2C(sda=Pin(5), scl=Pin(4))
LCD  = LCD1602(i2c,0x27)

# Defina os pinos dos sensores
termistor_pin= 15
ldr_pin  =  12
botao =Pin(14)

# Configuração dos ADCs para leitura dos sensores
adc_termistor = ADC(Pin(termistor_pin))
adc_ldr = ADC(Pin(ldr_pin))

# Função para leitura do valor do termistor
def ler_termistor():
    valor_adc = adc_termistor.read()
    tensao = valor_adc / 4095 * 3.3
    resistencia = (3.3 - tensao) / tensao * 10000
    temperatura = 1 / (1/298.15 + 1/3950 * log(resistencia/10000)/3950)
    temperatura -= 273.15
    return temperatura

# Função para leitura do valor do LDR
def ler_ldr():
    valor_adc = adc_ldr.read()
    return valor_adc


# Função para ler o valor dos sensores e exibir no display
def atualiza_display():
    temperatura = ler_termistor()
    luminosidade = ler_ldr()
    LCD.puts("Temp: {:.1f} C\n".format(temperatura),0,0)
    LCD.puts("Lum.: {}\n".format(luminosidade),1,1)

# Loop principal do programa
while True:
    # Verifique se o botão foi pressionado
    if botao.value() == 0:
        # Desligue o display
        LCD.backlight(False)
        sleep_ms(500)
    else:
        # Ligue o display e atualize os valores dos sensores
        LCD.backlight(True)
        atualiza_display()
    sleep_ms(100)
