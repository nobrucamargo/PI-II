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
Vamos tratar agora da lógica de como o código final funciona(clique [***aqui***](https://github.com/nobrucamargo/PI-II/blob/main/codigo_prog.md) para ver o código final) e como foi unificado todos os códigos de teste descritos na aba [***DESIGN***](https://github.com/nobrucamargo/PI-II/blob/main/design.md).
### 2.1 Junção dos códigos testes
Definido as portas de I/O do Arduíno e incluído as bibliotecas necessárias, foi utilizado uma lógica de programação onde as funções são executadas periódicamente dentro da função loop(), mas com o controle de um temporizador nas funções necessárias. Basicamente, as funções apenas checam os sensores e comandos seriail buscando alguma interação para habilitar, ou não, algum comando ou tarefa, portanto, nenhuma função fica esperando uma interação para executar algo, ao invés disso, há uma checagem frequênte da existência de interações. Concluindo, nenhuma função trava a outra, visto que nenhuma função fica no aguardo até que algo aconteça, exceto pela função lock(), que será explicada mais adinte. Veja abaixo um trexo do [***código final***](https://github.com/nobrucamargo/PI-II/blob/main/codigo_prog.md), onde a temporização de uma função é descrita:
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
### 2.3 função comandos_serial()


