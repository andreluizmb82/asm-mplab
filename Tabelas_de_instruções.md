### Considerações:

As imagens abaixo são formas didaticas de imaginarmos uma memoria de 8 bits.

<img src="./img/mem1.png" width="444" height="192" alt="Imagem">

<img src="./img/mem2.png" width="178" height="356" alt="Imagem">

<img src="./img/mem3.png" alt="Imagem">


Para facilitar a compreensão, ao ler registrador, pence em um espaço de memoria de 8 bits (1 Byte), que pode ser lida ou escrita por meio de *Instruções Orientadas a Byte*.

Cada bit pode guardar o valor lógico 1 ou 0.

Eletricamente o valor lógico 1 representa o sinal eletrico de nível alto (5V), e o valor lógico 0 representa um sinal eletrico de nível baixo(0V).

### 1. Tabela relacionando lógica com eletrica:
| Valor Lógico | Estado do Bit | Sinal Elétrico   | Ação Prática (Exemplo: LED) |
| :----------- | :------------ | :--------------- | :-------------------------- |
| 1            | Set (Setado)  | Nível Alto (5V)  | LED Ligado                  |
| 0            | Clear (Limpo) | Nível Baixo (0V) | LED Desligado               |


### 2. Instruções Orientadas a Bit (f = registrador, b = bit)
| Instrução    | Descrição da Instrução                 | Exemplo        | Descrição do Exemplo                                         |
| :----------- | :------------------------------------- | :------------- | :----------------------------------------------------------- |
| **BSF f, b** | Bit Set f -> Atribui 1 ao bit b em f   | `BSF PORTB, 1` | Atribui 1 ao bit 1 do registrador PORTB (Liga o pino RB1)    |
| **BCF f, b** | Bit Clear f -> Atribui 0 ao bit b em f | `BCF PORTB, 0` | Atribui 0 ao bit 0 do registrador PORTB (Desliga o pino RB0) |

### 3. Instruções Orientadas a Byte (f = registrador, d = destino, L = literal)
| Instrução       | Descrição da Instrução                        | Exemplo           | Descrição do Exemplo                                                      |
| :-------------- | :-------------------------------------------- | :---------------- | :------------------------------------------------------------------------ |
| **MOVLW L**     | Move valor Literal para registrador W         | `MOVLW 0x07`      | Move o valor hexadecimal 07 para o registrador de trabalho W              |
| **MOVWF f**     | Move o valor de W para o registrador f        | `MOVWF TRISB`     | Move o valor contido em W para o registrador TRISB                        |
| **DECFSZ f, d** | Decrementa f, pula próxima instrução se for 0 | `DECFSZ tempo, F` | Decrementa a variável tempo e salva o resultado nela mesma; pula se zerar |
| **GOTO k**      | Desvio incondicional para o rótulo k          | `GOTO loop`       | Salta o fluxo de execução para a linha identificada como 'loop'           |
| **CALL k**      | Chama sub-rotina no rótulo k                  | `CALL atraso`     | Desvia para a sub-rotina 'atraso' salvando o endereço de retorno          |
| **RETURN**      | Retorna de uma sub-rotina                     | `RETURN`          | Finaliza a sub-rotina e volta para a instrução logo após o CALL           |

### 4. Tabela de Diretivas
| Termo        | Descrição do Termo                                                                   |
| :----------- | :----------------------------------------------------------------------------------- |
| **#include** | Incorpora arquivos externos, como definições de nomes de registradores do chip       |
| **__CONFIG** | Define os bits de configuração de hardware (fuses) do microcontrolador               |
| **CBLOCK**   | Inicia um bloco de declaração de variáveis em endereços sequenciais da RAM           |
| **ENDC**     | Encerra o bloco de declaração de variáveis (CBLOCK)                                  |
| **ORG**      | Define o endereço de memória de programa onde as instruções seguintes serão gravadas |
| **END**      | Informa ao compilador o fim definitivo do arquivo de código fonte                    |


### 5. Programa:

```asm
; --- PROGRAMA PISCA LED (ESTILO CLÁSSICO) ---
#include <p16f628a.inc>

; Configuração do Hardware (Fuses)
 __CONFIG  _INTOSC_OSC_NOCLKOUT & _WDT_OFF & _PWRTE_ON & _LVP_OFF

    CBLOCK 0x20		; Início da memória RAM livre
        tempo		; Reserva um endereço de memória para a variável tempo
    ENDC

    ORG 0x00		; Endereço físico de Reset (Onde o PC aponta ao ligar)
    GOTO inicio

    ORG 0x05		; Pula o vetor de interrupção
inicio:			; Rotina de configuração (setup)
    ; Ir para Banco 1
    BCF     STATUS, IRP ; Reseta bit IRP (Vai para o Banco 1)
    BCF     STATUS, RP1 ; Reseta bit RP1 (Vai para o Banco 1)
    BSF     STATUS, RP0 ; Seta bit RP0 (Vai para o Banco 1)
    
    ; Configuração de Portas
    ; Configurando pinos do PORTB como saída
    MOVLW   B'00000000' ; Carrega W com zero (saída)
    MOVWF   TRISB       ; Move W para o registrador de direção (PORTB = Saída)
    
    ; Ir para Banco 0
    BCF     STATUS, RP0 ; Reseta bit RP0 (Volta para o Banco 0)
    
    ; Desabilitando compportamentos analogicos de pinos 
    MOVLW   0x07        ; Valor para desligar comparadores analógicos
    MOVWF   CMCON       ; Garante que os pinos sejam digitais
    
    ; Inicializando estado de pinos para observação no SFR-PORTB
    BSF     PORTB, 1    ; Coloca 5V no pino RB1 (Seta bit 1)
    BCF     PORTB, 2    ; Coloca 0V no pino RB2 (Seta bit 2)

loop:			; Rotina principal
    CALL    set_tempo
    BSF     PORTB, 0    ; Coloca 5V no pino RB0 (Seta bit 0)
    CALL    atraso      ; Chama sub-rotina de tempo
    
    CALL    set_tempo
    BCF     PORTB, 0    ; Coloca 0V no pino RB0 (Limpa bit 0)
    CALL    atraso
    
    GOTO    loop        ; Volta para o início do laço

set_tempo:
    MOVLW   0xff        ; Move 255 para W
    MOVWF   tempo       ; Move 7 de W para tempo
    RETURN
    
atraso:                 ; Sub-rotina simples de delay
    DECFSZ  tempo, F    ; Decrementa 'tempo', pula se for zero
    GOTO    atraso
    RETURN

    END                 ; Fim do arquivo (obrigatório)
```