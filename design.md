# *DESIGN* do projeto

Nessa etapa do projeto, foi planejado a organização e as conexões em geral do hardware, assim como tabelado o material necessário.
Para o projeto em questão, planeja-se usar o seguinte hardware:

**HARDWARE**    | **UNIDADES**
:------------:  | :------------:
Arduíno MEGA    | 1
Fonte 12 Vcc    | 1
LED             | 5
Resistor 220 ohm| 5
Matriz de LED's 8x8 com MAX7219| 1
Sensor de gás MQ-2| 1
Sensor de preseça PIR | 1
Servo motor     | 1

Agora, vejamos abaixo uma abstração geral do design do projeto:
 ![design](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/design.png)
> >*Descrição:* Os fios vermelhos e pretos representam a alimentação de 5V e terra, respectivamente, enquanto os fios amarelos e verde representam os sinais 
> >digitais e analógicos(I/O), respectivamente. 
> 
>- ![arduíno](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/arduino.png)O Arduíno é a plataforma de controle escolhida, que alimentará, receberá e enviará sinais digitais e analógico para os shields.
>- ![leds](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/led.png)Os LED's brancos representam a iluminação do apartamento, que receberá sinais digitais de comando para acender ou apagar. A corrente é limitada por um resistor de 150ohm;
>- ![motor](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/motor.png)O Servomotor receberá um sinal digital para abrir ou fechar a persiana do quarto;
>- ![matriz](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/matriz.png)A matriz de LED's representa a fita de LED da sala de estar e receberá 3 sinais digitais de controle;
>- ![gas](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/gas.png)O sensor de gás envia um sinal digital e um analógico assim que detecta vazamento de gás;
>- ![presenca](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/presenca.png)O sensor de presença envia um sinal digital assim que detecta algum tipo de movimentação;
>- ![fonte](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/fonte.png)As baterias representam uma possível fonte auxiliar de alimentação;
