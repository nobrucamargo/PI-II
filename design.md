# *DESIGN* do projeto

Nessa etapa do projeto, foi planejado a organização e as conexões em geral do hardware, assim como tabelado o material necessário.
Veja abaixo uma abstração geral do design do projeto:
> ![design](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/design.png)
> >*Descrição:* Os fios vermelhos e pretos representam a alimentação de 5V e terra, respectivamente, enquanto os fios amarelos e verde representam os sinais 
> >digitais e analógicos, respectivamente, de envio e recebimento. 
> 
>- ![arduíno](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/arduino.png)O Arduíno é a plataforma de controle escolhida, que alimentará, receberá e enviará sinais digitais e analógico para os shields.
>- ![leds](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/led.png)Os LED's vermelhos representam a iluminação do apartamento, que receberá sinais digitais de comando para acender ou apagar. A corrente é limitada por 
> um resistor;
>- ![motor](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/motor.png)O Servomotor receberá um sinal digital para abrir ou fechar a persiana do quarto;
>- A matriz de LED's representa a fita de LED da sala de estar e receberá 3 sinais digitais de controle;
>- O sensor de gás envia um sinal digital e um analógico assim que detecta vazamento de gás;
>- O sensor de presença envia um sinal digital assim que detecta algum tipo de movimentação;
>- As baterias representam uma possível fonte auxiliar de alimentação;
