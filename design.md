# *DESIGN* do projeto

## 1. Hardware
Nessa etapa do projeto, foi planejado a organização e as conexões em geral do hardware, assim como tabelado o material necessário.
Para o projeto em questão, planeja-se usar o seguinte hardware:

**HARDWARE**    | **UNIDADES**
:------------:  | :------------:
Arduíno MEGA    | 1
Fonte 12 Vcc    | 1
LED             | 5
Resistor 470 ohm| 5
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

## 2. Software
Estudado o funcionamento dos shields do Arduíno(Hardware), parte-se então, para os testes de funcionamento de cada shield. Com o intuito de maximizar a produtividade e minimizar o tempo, os shields foram testados a medida que foram programados no código central do projeto, por tanto, aqui está adiantado algumas partes do programa final, constante na próxima etapa, a implementação. Os pinos foram definidos de forma a manterem concordância com os pinos usados no design do hardware.
> Clique [*aqui*](https://github.com/nobrucamargo/PI-II/blob/main/codigo/programa_pi.ino) para visualização completa do código;
### 2.1 Sensor de gás
O sensor de gás funcionou como o esperado, mas deve-se atenar ao fato de o sensor mandar sinal baixo (LOW) caso detecte gás, e alto(HIGH) caso não detecte gás. A função int gas() descrita abaixo foi utilizada como teste e será útil para uma função que bloqueie o funcionamento de qualquer dispositivo que possa provocar uma explosão com faíscas. Para melhor entendimento do objetivo dessa função, veja o resto das funções e depois, analise-as todas juntas no código completo.
~~~C++
...
  /*gas: Verifica se tem gás ou não*/
int gas(){
  if (digitalRead(FUMACA)==HIGH) return 0;
  else return 1;
}
...
~~~
### 2.2 Iluminação e sensor de presença
Os leds e o sensor de presença funcionaram como esperado, no entanto, constatou-se que o brilho dos led's estavam fracos, portanto, recomenda-se a alteração dos resistores para de menor resistência, como resistores de 390 ohm.
Os leds e o sensor de presença foram testados conforme o código da função void lampadas() descrita abaixo, onde um vetor foi criado para simular os interruptores.
~~~C++
...
bool interruptor[5]={false, false, false, false, false};        //Vetor de interruptores do apartamento;
...
/* lampadas: liga/desliga lâmpadas com base nos interruptores/presença*/
void lampadas() {

  unsigned long tempo_atual = millis();

  /* Hora de verificar interruptores, mais informações no código completo */
  if (tempo_atual - tempo_tarefa_1>periodo_tarefa_1) {
    tempo_tarefa_1 = tempo_atual;

    //liga ou desliga led se interruptor alterado
    if (interruptor[0] == true) {
      digitalWrite(LED_ROOM, HIGH);
    }
    else {
      digitalWrite(LED_ROOM, LOW);
    }
    
    if (interruptor[1] == true) {
      digitalWrite(LED_KITCHEN, HIGH);
    }
    else {
      digitalWrite(LED_KITCHEN, LOW);
    }

    if (interruptor[2] == true) {
      digitalWrite(LED_LIVING_ROOM, HIGH);
    }
    else {
      digitalWrite(LED_LIVING_ROOM, LOW);
    }

    if (interruptor[3] == true) {
      digitalWrite(LED_WC, HIGH);
    }
    else {
      digitalWrite(LED_WC, LOW);
    }
    
    //Se houver presença no corredor ou se o interruptor for alterado;
    if (interruptor[4] == true || digitalRead(PRESENCA)==HIGH) {
      digitalWrite(LED_HALL, HIGH);
    }
    else {
      digitalWrite(LED_HALL, LOW);
    }
  }
}
~~~
## 2.3 Matriz de leds(fita de leds)
A matriz de leds funcionou como esperado. A função descrita abaixo controla o estado e a intensidade de um "L" mostrado no display da matriz, simulando a fita de leds descrita na maquete eletrônica.
~~~C++
...
#include "LedControl.h"     //para controlar a matriz de leds(fita de leds);

bool interruptor_led=false, aumenta_intensidade=false, diminui_intensidade=false;
int intensidade_padrao=4;
...
void setup() {

  /*Inicia a matriz de leds sem modo de economia de energia(com os leds habilitados)*/
  matrix.shutdown (0 , false);  //0=índice da matríz em cascata; Se houvesse outra, seu índice seria 1, e assim por diante
  /*limpa display da matrix de leds*/
  matrix.clearDisplay(0);
  /*intensidade padrão da fita*/
  matrix.setIntensity(0, intensidade_padrao);
}
...
/*fita_leds: gerencia a fita de leds, conforme interruptor_led e demais comandos;*/
void fita_leds(){
  
  unsigned long tempo_atual = millis();
  if(tempo_atual-tempo_led>periodo_tarefa_1){
    tempo_led=tempo_atual;
    
    /*liga/desliga a fita de led*/
    if (interruptor_led == true){
      /*liga fita de led*/
      matrix.setRow(0, 7, B11111111);
      matrix.setColumn(0, 7, B11111111);
    }
    else 
      matrix.clearDisplay(0);

    /*controle de intensidade da fita de led*/
    if (aumenta_intensidade==true){
      matrix.setIntensity(0, ++intensidade_padrao);
      aumenta_intensidade=false;
      
    }
    if (diminui_intensidade==true){ 
      matrix.setIntensity(0, --intensidade_padrao);
      diminui_intensidade=false;
    }
  }
}
~~~
## 2.4 Servo motor
O servo motor não funcionou como esperado. Foi necessário um bom estudo de caso sobre o dispositivo, através de testes e pesquisa do funcionamento, afim de aplicar corretamente o seu uso. Usando o código descrito abaixo, foi testado diversos ângulos diferentes, até ser encontrado os ângulos corretos a serem usados.
~~~C++
...
#include <VarSpeedServo.h>  //para controlar o servo motor com velocidade variável;
...
VarSpeedServo motor;    //Criando objeto para o servo motor;...
void setup() {
motor.attach(SERVO); //Associando o pino correto ao objeto motor;
}
...
/* servo: Abre/ fecha as persianas, conforme comando de abrir/fechar*/
void servo(){
  
  unsigned long tempo_atual=millis();
  if(tempo_atual-tempo_servo>periodo_tarefa_1){
    tempo_servo=tempo_atual;
    
    /*Abre/fecha janela conforme sinal de entrada*/
    if(abrir==true){
      motor.write(170, 30); //Gira o motor em 90° a uma velocidade de 60.
                           //velocidade 1 é a mínima, 255 é a máxima e 0 é a padrão.
      abrir=false;
    }
    if(fechar==true){      //Análogo ao anterior, só que agora para fechar a janela
      motor.write(0, 60);
      fechar=false;
    }
  }
}
~~~
