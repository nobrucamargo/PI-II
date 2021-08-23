# **CÓDIGO DE PROJETO** 
Aqui se encontra o código criado para a implementação do projeto, completo e detalhado por partes.
## 1. Código completo:
```C++
//Definindo as portas de I/O do arduíno
#define LED_ROOM 41
#define LED_KITCHEN 43
#define LED_LIVING_ROOM 45
#define LED_WC 47
#define LED_HALL 49
#define PRESENCA 51
#define FUMACA 53
#define SERVO 23

#include "LedControl.h"     //para controlar a matriz de leds(fita de leds);
#include <VarSpeedServo.h>  //para controlar o servo motor com velocidade variável;

/* Essas variáveis são globais pois é necessário
   manter os valores indenpendente do contexto de
   execução das funçôes chamadas no loop()*/
const unsigned long periodo_tarefa_1 = 1000;        //tarefa_1 caso seja necessário definir outro período no futuro;
unsigned long tempo_tarefa_1 = 0, tempo_gas=0, tempo_led=0, tempo_servo=0;           
bool interruptor[5]={false, false, false, false, false};        //Vetor de interruptores do apartamento;
bool interruptor_led=false, aumenta_intensidade=false, diminui_intensidade=false;
bool abrir=false, fechar=false;
int intensidade_padrao=4, modo=1; //intensidade padrão da fita de led, modo parão para a matriz de leds não piscar;

LedControl matrix=LedControl(25, 27, 29, 1); //Pino23: DIN;
                                             //Pino25: CLK;
                                             //Pino27: CS;
                                             //4° Elemento (1): Quantidade de módulo-matrizes em cascata;
VarSpeedServo motor;    //Criando objeto para o servo motor;

void setup() {

  /* Comunicação serial com o computador */
  Serial.begin(9600);
  while (!Serial);

  /* Configuração dos pinos como entrada ou saída */
  pinMode(LED_ROOM, OUTPUT);
  pinMode(LED_KITCHEN, OUTPUT);
  pinMode(LED_LIVING_ROOM, OUTPUT);
  pinMode(LED_WC, OUTPUT);
  pinMode(LED_HALL, OUTPUT);
  pinMode(PRESENCA, INPUT);
  pinMode(FUMACA, INPUT);

  /*Inicia a matriz de leds sem modo de economia de energia(com os leds habilitados)*/
  matrix.shutdown (0 , false);  //0=índice da matríz; Se houvesse outra, seu índice seria 1, e assim por diante
  /*limpa display da matrix de leds*/
  matrix.clearDisplay(0);
  /*intensidade padrão da fita*/
  matrix.setIntensity(0, intensidade_padrao);

  motor.attach(SERVO); //Associando o pino correto ao objeto motor;
}

/*gas: Verifica se tem gás ou não*/
int gas(){
  if (digitalRead(FUMACA)==HIGH) return 0;
  else return 1;
}

/*lock: Trava de segurança - Para tudo enquanto houver sinal de fumaça/gás, e emite um alerta a cada 5 seg*/
void lock(){
   
   /*hora de verificar o gás*/
  unsigned long tempo_atual=millis();
  tempo_gas=tempo_atual-tempo_gas;
  if (tempo_gas>periodo_tarefa_1){
    tempo_gas=tempo_atual;

    int i=0; //Contador para envio sucessivo de alertas;
    while (gas()){
    
      if (i==1){
        Serial.print("ALERTA!! GÁS DETECTADO!! ");
        i=0;
      }
    //Mantém tudo parado, em nível lógico baixo;  
    digitalWrite(LED_ROOM, LOW);
    digitalWrite(LED_KITCHEN, LOW);
    digitalWrite(LED_LIVING_ROOM, LOW);
    digitalWrite(LED_WC, LOW);
    digitalWrite(LED_HALL, LOW);
    matrix.clearDisplay(0);
    motor.stop();
    delay(5000); //Delay de 5s para envio de alerta caso ainda haja sinal de gás;
    i++;
    }
  }
}


/* lampadas: liga/desliga lâmpadas com base nos interruptores/presença*/
void lampadas() {

  unsigned long tempo_atual = millis();

  /* Hora de verificar interruptores */
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
      default :
         break;
    }
    
    /*switch não suporta mais que 8 cases, então fiz outro*/
    switch (dado_recebido){
      case '+':
         aumenta_intensidade=true;
         break;
      case '-':
         diminui_intensidade=true;
         break;
      case '1':
         modo=1;
         break;
      case '2':
         modo=2;
         break;
      case '3':
         modo=3;
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

/*fita_leds: gerencia a fita de leds, conforme interruptor_led e demais comandos;
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

/* servo: Abre/ fecha as persianas, conforme comando de abrir/fechar*/
void servo(){
  
  unsigned long tempo_atual=millis();
  if(tempo_atual-tempo_servo>periodo_tarefa_1){
    tempo_servo=tempo_atual;
    
    /*Abre/fecha janela conforme sinal de entrada*/
    if(abrir==true){
      motor.write(90, 30); //Gira o motor em 90° a uma velocidade de 60.
                           //velocidade 1 é a mínima, 255 é a máxima e 0 é a padrão.
      abrir=false;
    }
    if(fechar==true){
      motor.write(-90, 60);
      fechar=false;
    }
  }
}

/*loop: Executa tudo*/
void loop(){
   lock();
   comandos_serial();
   fita_leds();
   servo();
   lampadas();
}
```
