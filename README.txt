
###################################### 16-bit processor with DOT acceleration #################################

#### Team ###
* João Coelho (me) 
* Sara Cruz
* Rita Vieira

### Final Grade ####
* 19/20

##################################################### READ ME #################################################
                                            ### Formatos de Instruções ###
As instruções têm um tamanho fixo de 16 bits e optamos por separá-las em três tipos: Tipo I (Imediato), Tipo A (Dois Registos) e Tipo D (Três Registos). A escolha do tamanho dos campos baseou-se nos seguintes critérios: 
* Registos (3 bits): Permite endereçar um banco de 8 registos (R0 a R7).
* Opcode (3 bits): Permite mapear até 8 instruções e, assim, há combinações livres para o caso de extensão da arquitetura. No Tipo I, usamos apenas o msb(bit 12) como 0, deixando os restantes bits para aumentar o imediato e, consequentemente, a expressividade da ISA. Nos tipos A e D, os 3 bits são usados para distinguir as operações aritméticas.
* Funct3 e funct4: Escolhemos criar estes campos com intuito de melhorar a extensibilidade da nossa arquitetura. Neste projeto são preenchidos com zeros, no entanto, caso haja necessidade, é possível não só serem substituídos por outros campos (p.e mais registos) como também por outras combinações, de forma a diferenciar novas funções.
                                                     Instrução                     Codificação
  15   13 12   11 10 9     7 6      4 3       0                          Tipo | Opcode | funct3 | funct4 |
I |  rd  | op |                  imm          |    li    rd, imm          I        0
A |  rd  |  opcode  |  rs1  | funct3 | funct4 |    add   rd, rs1          A       100     000      0000
D |  rd  |  opcode  |  rs1  |  rs2   | funct4 |    dot   rd, rs1          A       101     000      0000
                                                   dota  rd, rs1, rs2     D       110              0000
                                                                                          
                                      ### Organização interna do datapath  ###
O fluxo do datapath é sempre linear (ROM - decodificação - banco de registos - ALU - write-back em rd)  tornando o desenho claro e direto. 
As instruções dot e dota partilham a mesma unidade multiplicadora e somadora de vetores, alternando-se apenas a origem dos operandos (rd e rs1 para o dot; rs1 e rs2 para o dota), através de dois multiplexers (um para o rd/rs2 outro para o rd+1/rs2+1) escolhe-se os operandos com base na instrução. Da mesma forma, o cálculo da soma de add e a acumulação final de dota utilizam o mesmo somador central para fazer R[rd] + R[rs1](add) ou R[rd] + (dot rs1, rs2) (dota) e, assim, o número de componentes é reduzido significativamente.
Por fim, a ordenação dos campos no barramento de saída da ROM e a disposição dos multiplexers no banco de registos foram escolhidas de forma a minimizar o cruzamento de linhas no circuito, tornando o desenho mais limpo e legível.

                                          ### Sinais de controlo ###
Os sinais de controlo são extraídos diretamente dos bits da instrução, cada bit do opcode liga diretamente a um MUX que controla, o que simplifica o circuito e elimina lógica combinatória adicional.
* O bit 12 controla o MUX de write-back, selecionando entre o imediato (bit 12 = 0) e o resultado da ALU (bit 12 = 1) para escrever em rd.
* O bit 11 controla o MUX antes do somador central, selecionando o que é somado a R[rd], se for add (bit 11 = 0), soma R[rs1] diretamente, se for dota(bit 11 = 1), soma o resultado do bloco de produto escalar.
* O bit 10 controla os dois MUXes nas entradas dos blocos multiplicadores, alternando a origem dos operandos. No dota (bit 10 = 0), os operandos são R[rs1]/R[rs1+1] e R[rs2]/R[rs2+1]. Já no dot (bit 10 = 1), passam a ser R[rd]/R[rd+1] e R[rs1]/R[rs1+1].

                                        ### Exemplos de Codificação ###
* li R1, 3                                                       * dot R0, R2
0010 0000 0000 0011 (0x2003)                                     0001 0101 0000 0000 (0x1500)
* add R3, R1                                                     * dota R0, R4, R6
0111 0000 1000 0000 (0x7080)                                     0001 1010 0110 0000 (0x1a60)
###############################################################################################################
