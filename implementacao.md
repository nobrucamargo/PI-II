# IMPLEMENTAÇÃO DO PROJETO
## 1. Maquete e instalação do hardware
Para a construção da maquete, foi tutilizado isopor, estilete, pistola de cola quente, bastão de cola quente, folhas coloridas, papelão e palitos de madeira pequenos e grandes. Primeiramente foi passado a planta do apartamento para um isopor e recortado as paredes com um estilete aquecido em fogo, depois foi colado folhas verdes como piso e, então, fixado as paredes com cola de isopor e palitos pequenos de madeira. Veja abaixo a vista superior do molde da maquete: 
> ![vista_superior_molde](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/imagem_1.jpeg)
> > Vista superior da base da maquete. 

As tinturas das paredes e moldes das portas foram feitos também com folhas coloridas.
A cortina foi feita com picotes de folhas, linha, polias de papelão, cola quente e palitos grandes de madeira. O servo motor foi acoplado às polias com um palito grande de madeira e cola quente, assim, ao girar, fecha ou abre as persianas conectadas por um fio a duas polias, uma fixada no palito e a outra livre para girar.
Os leds, a matriz de leds, o sensor de presença e o sensor de gás foram instalados no chão da maquete, afim de melhorar a vizualização da implementação, visto que não há telhado na maquete. Veja abaixo a vista superior da maquete com hardware instalado: 
> ![vista_superior](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/imagem_superior.jpeg)
> > Vista superior da maquete pronta.

Para a instalção do hardware, foi feito uma pequena área suspensa debaixo da maquete, afim de depositar o arduíno e a protoboard auxiliar necessária para a conexão da fiação elétrica, que, por sua vez, foi feita por debaixo da maquete. Veja abaixo uma imagem da vista inferior da maquete: 
> ![vista_inferior](https://github.com/nobrucamargo/PI-II/blob/main/Imagens/imagem_inferior.jpeg)
> > Vista inferior da maquete, contendo a fiação elétrica.

Note que algumas modificaçõe foram feitas apartir da planta da maquete. Foi criado uma mesa entre a sala e a cozinha, e foi criado uma janela no corredor, tudo com o objetivo de tornar melhor a vizualização da implementação do sistema.

## 2. Código final
Vamos tratar agora da lógica de funcionamento do código final(clique [***aqui***](https://github.com/nobrucamargo/PI-II/blob/main/codigo/programa_pi.ino) para ver o código final) e como foi unificado todos os códigos de testes descritos na aba [***DESIGN***](https://github.com/nobrucamargo/PI-II/blob/main/design.md). Juntamente com a explicação da lógica, está contido alguns traços do código final, afim de contextualizar o leitor, mas não é extritamente necessário a leitura e interpretação desses traços.
### 2.1 Junção dos códigos testes
Definido as portas de I/O do Arduíno e incluído as bibliotecas necessárias, foi utilizado uma lógica de programação onde as funções são executadas periódicamente dentro da função loop(), mas com o controle de um temporizador nas funções necessárias. Basicamente, as funções apenas checam os sensores e comandos seriail buscando alguma interação para habilitar, ou não, algum comando ou tarefa, portanto, nenhuma função fica esperando uma interação para executar algo, ao invés disso, há uma checagem frequênte da existência de interações. Concluindo, nenhuma função trava a outra, visto que nenhuma função fica no aguardo até que algo aconteça, exceto pela função lock(), que será explicada mais adinte. Veja abaixo um trexo do [***código final***](https://github.com/nobrucamargo/PI-II/blob/main/codigo/programa_pi.ino), onde a temporização de uma função é descrita:
~~~C++
...
/* Essas variáveis são globais pois é necessário
   manter os valores indenpendente do contexto de
   execução das funçôes chamadas no loop()*/
const unsigned long periodo_tarefa_1 = 1000; //tarefa_1 caso seja necessário definir
                                             //um período diferente para a execução de outra tarefa;
unsigned long tempo_gas=0;
...
void lock(){
   
   /*hora de verificar o gás*/
  unsigned long tempo_atual=millis();
  tempo_gas=tempo_atual-tempo_gas;
  if (tempo_gas>periodo_tarefa_1){
    tempo_gas=tempo_atual;
...
~~~
Desta forma, foi possível juntar todas as funções na função loop(), sem uso da função delay(), que impede que as outras interações/comandos sejam verificados em um período apropriado.
### 2.2 Função lock() - Exceção
A função lock() é a função encarregada de bloquear o funcionamento de todos os outros dispositivos caso seja detectado vazamento de gás, afim de evitar faíscas e possíveis explosões. Além disso, ela emite um sinal de alerta no display do computador de monitoramento de segurança a cada 5 segundos. Portanto podemos considerar essa função como sendo a função máxima, já que, caso haja sinal de gás, nenhum outro comando ou interação importa e, por isso, apenas para essa função há a exceção do uso consciente da função delay(), aqui utilizada para diminuir a frequência de checagem do gás e enviar um alerta para o monitoramento apenas a cada 5s. Veja abaixo a função lock() completa:
~~~C++
/*lock: Trava de segurança - Para tudo enquanto houver sinal de fumaça/gás, e emite um alerta a cada 5 seg*/
void lock(){
   
   /*hora de verificar o gás*/
  unsigned long tempo_atual=millis();
  tempo_gas=tempo_atual-tempo_gas;
  if (tempo_gas>periodo_tarefa_1){
    tempo_gas=tempo_atual;

    int i=0; //Contador para envio sucessivo de alertas;
    while (gas()){
   
    Serial.print("ALERTA!! GÁS DETECTADO!! ");

    //Mantém tudo parado, em nível lógico baixo;  
    digitalWrite(LED_ROOM, LOW);
    digitalWrite(LED_KITCHEN, LOW);
    digitalWrite(LED_LIVING_ROOM, LOW);
    digitalWrite(LED_WC, LOW);
    digitalWrite(LED_HALL, LOW);
    matrix.clearDisplay(0);
    motor.stop();
    delay(5000); //Delay de 5s para envio de alerta caso ainda haja sinal de gás;
    }
  }
}
~~~
### 2.3 Função comandos_serial()
Na aba [***Design***](https://github.com/nobrucamargo/PI-II/blob/main/design.md) foi explicado o funcionamento da função lampadas(), mas para a integração dessa função ao código final, assim como a integração das demais funções que necessitam de comandos seriais, foi criado a função comandos_serial(). A função comados_serial() apenas gerencia os comandos seriais, simulando interruptores para as lâmpadas e para a fita deleds, e tamém simulando pushbuttons para o controle da intensidade do brilho da fita de leds e a abertura ou fechamento das persianas. Veja abaixo a função comandos_serial():
~~~C++
/* comandos_serial: gerencia os comandos da porta serial */
void comandos_serial() {

  /* Caso tenha recebido algum dado do PC */
  if (Serial.available()) {
    char dado_recebido = Serial.read();

  /*gerencia comandos das lâmpdas, fita de leds e servo motor*/
    switch (dado_recebido){
      case 'r':
         if (interruptor[0] == true)
          interruptor[0] = false;
         else
          interruptor[0] = true;
         break;
      case 'k':
         if (interruptor[1] == true)
          interruptor[1] = false;
         else
          interruptor[1] = true;
         break;
      case 'l':
         if (interruptor[2] == true)
          interruptor[2] = false;
         else
          interruptor[2] = true;
         break;
      case 'w':
        if (interruptor[3] == true)
          interruptor[3] = false;
        else
          interruptor[3] = true;
         break;
      case 'h':
        if (interruptor[4] == true)
          interruptor[4] = false;
        else
          interruptor[4] = true;
         break;
      case 'f':
         if (interruptor_led==true)
          interruptor_led=false;
         else 
          interruptor_led=true;
         break;
      case '+':
         aumenta_intensidade=true;
         break;
      case '-':
         diminui_intensidade=true;
         break;
      case 'o':
         abrir=true; 
         break;
      case 'c':
         fechar=true;
         break;
      default: 
         break;
    }   
  }
}
~~~

