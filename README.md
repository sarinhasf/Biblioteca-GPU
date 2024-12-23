<div align="center">  
  <img align="center" width=100% src="https://github.com/sarinhasf/Biblioteca-GPU/blob/main/img/capa.png" alt="GraphLib">
</div>

<h4 align="center">Projeto da disciplina TEC 499 - Sistemas Digitais - TP04 | Grupo 02</h4>

## Sumário
- [Visão Geral do Projeto](#Visão-Geral-do-Projeto)
- [Requisitos](#Requisitos)
- [Arquitetura do Kit de Desenvolvimento DE1-SoC](#Arquitetura-do-Kit-de-Desenvolvimento-DE1-SoC)
- [Sobre o processador utilizado](#Sobre-o-Processador-Gráfico-Utilizado)
- [Desenvolvimento da Biblioteca](#Desenvolvimento-da-Biblioteca)
- [Como usar a GraphLib?](#Como-usar-a-GraphLib)
- [Testes](#Testes)
- [Tecnologias e Ferramentas utilizadas](#Tecnologias-e-Ferramentas-utilizadas)
- [Configurações de Ambiente e Execução](#Configurações-de-Ambiente-e-Execução)
- [Desenvolvedoras](#Desenvolvedoras)
- [Referências](#Referências)


# Visão Geral do Projeto
O objetivo central do problema 2 foi criar uma biblioteca em assembly com funções gráficas para poder manipular a GPU feita pelo estudante Gabriel Sá Barreto, para assim, facilitar a implementação do jogo Tetris feito no problema 1. No momento, essa GPU se encontra na placa que estamos utilizando para desenvolver o problema, a DE1-SOC.

O trabalho de Conclusão de Curso de Gabriel pode ser acessado em: https://drive.google.com/file/d/1MlIlpB9TSnoPGEMkocr36EH9-CFz8psO/view


# Requisitos 
- Desenvolver uma biblioteca em Assembly para manipular as funções gráficas do Processador Gráfico feito por Gabriel Sá Barreto. 
- Funcionalidades da biblioteca:
  - Exibir sprites salvos da memória
  - Desenhar quadrados de tamanho 8x8
  - Desenhar poligonos (quadrados e triângulos)
  - Modificar sprites da memória 
  - Definir a cor base do background


# Arquitetura do Kit de Desenvolvimento DE1-SoC
O Kit de Desenvolvimento DE1-SoC ostenta uma plataforma de design de hardware robusta com base no FPGA System-on-Chip (SoC) da Altera, que combina núcleos embarcados dual-core Cortex-A9 com lógica programável, oferecendo máxima flexibilidade de design. Dispondo do poder da reconfigurabilidade aliado a um sistema de processador de alto desempenho e baixo consumo de energia. 

O System-on-Chip (SoC) da Altera integra um sistema de processador (HPS) baseado em ARM, composto por processador, periféricos e interfaces de memória conectados de forma integrada à estrutura FPGA, utilizando interconexão de alta largura de banda. Incluindo hardwares como memória DDR3 de alta velocidade, recursos de áudio e vídeo, rede Ethernet, entre outros.

<div align="center">  
  <img align="center" width=60% src="https://github.com/naylane/candi-block/blob/main/img/kit%20de1soc.png" alt="Placa DE1-Soc">
  <p><em>Placa de Desenvolvimento DE1-SoC</em></p>
</div>

### Diagrama de Blocos da Placa DE1-SoC
Para que os usuários desfrutem de máxima flexibilidade, todas as conexões são realizadas através do dispositivo Cyclone V SoC FPGA, proporcionando liberdade ao configurar o FPGA para implementar os mais diversos projetos de sistema.

<div align="center">  
  <img align="center" width=60% src="https://github.com/naylane/candi-block/blob/main/img/diagrama.png" alt="Diagrama de Blocos DE1-Soc">
  <p><em>Diagrama de Blocos</em></p>
</div>


# Sobre o Processador Gráfico Utilizado
A GPU da placa DE1-SOC que estamos utilizando para a elaboração dos projetos do PBL de SD foi substituída por um um processador gráfico feito pelo aluno Gabriel Sá Barreto em seu TCC de tema "Desenvolvimento de uma Arquitetura Baseada em Sprites para criac ̧ao de Jogos 2D em Ambientes Reconfiguraveis utilizando dispositivos FPGA", material este que usamos como base durante todo projeto. 
Esse processador permite mover e controlar elementos em um monitor VGA com resolução de 640x480 pixels. Dessa forma, ele contém funções que permite desenhar polígonos convexos (Quadrado e Triângulo), desenhar sprits, além de pintar o background ou uma parte específica do background. 

## Arquitetura do Processador Gráfico 
- Falando um pouco sobre sua arquitetura, ela consiste em um processador de propósito geral, duas FIFOs (First In First Out), uma PLL (Phase Locked Loop) e um Processador Gráfico. Para atuar como processador de propósito geral foi escolhido o Nios II. 
- O Nios II armazena nas FIFOs todas as instruções que devem ser executadas pelo Processador Gráfico. O processador Nios II controla a lógica geral do sistema, ele envia comandos de controle (inst_a, inst_b) e escreve dados para a FIFO A através do sinal wrfull, e também recebe informações de status, como o sinal de reset e clock. 
- É importante ressaltar que o controle de acesso para leitura/escrita é realizado pelo controlador de barramento Nios II. Assim, é realizado a distribuição dos campos das intruções do Processador Gráfico dentro dos barramentos dataA e dataB no momento do envio, assim mantendo esses dois barramentos de 32-bits separados podemos construir instruções de até 64-bits.
- Em geral, o barramento dataA é utilizado para opcodes e endereçamento do Banco de Registrador e Memórias, enquanto o barramento dataB é utilizado para envio dos dados a serem armazenados ou modificados.
- Após a inserção das instruções nos barramentos, deve-se colocar o sinal do wrreg (sinal de start) em nível lógico alto, pois ele irá ativar o Móduolo gerador de pulso no qual irá habilitar a escrita nos FIFOs. Lopo após, deve-se novamente por em nível lógico baixo, pois assim reiniciamos o módulo gerador de pulso.

<div align="center">  
  <img align="center" width=50% src="https://github.com/sarinhasf/Biblioteca-GPU/blob/main/img/arq.png" alt="Arquitetura">
  <p><em>Arquitetura do processador gráfico do Gabriel de Sá Barreto</em></p>
</div>

## Instruções do Processador Gráfico 
O processador gráfico do Gabriel contém algumas instruções para exibir elementos gráficos no VGA, elas são:
  1) Escrita no Banco de Registradores (WBR): Essa instrução é responsável por configurar os
  registradores que armazenam as informações dos sprites e a cor base do background.
  2) Escrita na Memória de Sprites (WSM): Essa instrução armazena ou modifica o conteúdo presente na Memória de Sprites.   
  3) Escrita na Memória de Background (WBM): Essa instrução armazena ou modifica o conteúdo presente na Memória de Background. Sua função é configurar valores RGB para o preenchimento de áreas do background. 
  4) Definição de um Polígono (DP): Essa instrução é utilizada para modificar o conteúdo da Memória de Instrução do Coprocessador, de forma a definir os dados referentes a num polígono que deve ser renderizado.

Assim, no nosso código em assembly, passando corretamente os parâmetros atráves dos barramentos dataA e dataB, e sempre habilitando e desabilitando o wrreg para o correto funcionamento das intruções, conseguimos construir a GraphLib (nossa biblioteca).

# Desenvolvimento da Biblioteca
GraphLib é uma biblioteca desenvolvida em assembly para interagir com o processador gráfico, citado anteriormente, para exibir elementos no monitor através do VGA. A biblioteca oferece um conjunto de funções especializadas para manipulação de elementos gráficos como sprites, background e polígonos, utilizando instruções de baixo nível para comunicação direta com o hardware. Foi implementado em todas as funções o devido salvamento e recuperação de contexto, além da desativação e ativação do wrreg quando necessário.

<div align="center">  
  <img align="center" width=50% src="https://github.com/sarinhasf/Biblioteca-GPU/blob/main/img/animacao_gpu.gif" alt="GraphLib">
  <p><em>Desenvolvimento da Biblioteca</em></p>
</div>

## Implementação das Instruções em Assembly
Apresentação do código assembly que compõe a biblioteca gráfica. Como as instruções WBR, WBM, WSM e DP foram implementadas, como os bits são manipulados para formar instruções e como os dados são enviados aos barramentos para execução pelo processador gráfico. 

### Organização Geral das Instruções
As instruções seguem a mesma estrutura básica, permitindo que sejam interpretadas e executadas pelo processador gráfico:
- Organização dos Bits:
Os bits de cada dado são organizados em campos específicos na Instruction Word (palavra de controle).
Cada parâmetro ocupa uma posição predefinida, utilizando deslocamentos de bits para garantir o alinhamento correto.
- Envio para os Barramentos:
Após montados, os bits são enviados para os barramentos dataA e dataB.   
- Ativação do wrreg:
O wrreg é ativado (1) para sinalizar a execução da instrução. Em seguida, ele é desativado (0) para permitir novas operações.

### Montagem das Instruções
A montagem das instruções no processador gráfico segue uma lógica baseada no deslocamento de bits. Esse processo organiza os dados em posições específicas (campos) em um registrador para montar a Instruction Word, palavra de controle da instrução, garantindo que cada instrução seja interpretada corretamente. 

As instruções possuem uma estrutura própria pré-definida, dividida em campos. Cada campo ocupa um intervalo de bits específico dentro da Instruction Word e o deslocamento é utilizado para posicionar cada dado alinhando em seu respectivo campo.

A montagem de uma instrução envolve três passos principais:
- Preenchimento dos Campos: Cada dado é deslocado para os bits do campo que deve ocupar na instrução, começando pelo mais significativo.
- Concatenação dos Campos: Os valores dos campos são combinados utilizando operações bit a bit (OR/ADD).
- Envio da Instrução: Após a montagem, a Instruction Word completa é enviada para execução através dos barramentos.
  
Dessa forma, todas as instruções são montadas e permitem a comunicação com o processador gráfico.

### Inicialização e Mapeamento de Memória
A função `inicializa_fpga` abre o arquivo `/dev/mem`, que permite acesso à memória física do sistema. Em seguida, configura o mapeamento de memória (mmap) para o processador gráfico, conectando a memória virtual à memória física. A Ponte entre Processador e FPGA, esta função é fundamental para o funcionamento da biblioteca, fazendo com que o código assembly tenha acesso aos registradores e às memórias do processador gráfico. Através do mapeamento, podemos escrever instruções diretamente na memória da FPGA, comandando o comportamento do hardware e renderizando gráficos na tela.

### Encerramento e Liberação da Memória
A função `fecha_dev_mem` é responsável por liberar a memória mapeada e fechar o arquivo `/dev/mem`. O código `munmap` desfaz o mapeamento de memória, liberando a memória virtual que havia sido conectada à memória física. O `close` fecha o arquivo `/dev/mem`, finalizando a conexão com a FPGA.

# Como usar a GraphLib
Para usar a GraphLib basicamente no seu código em C você vai chamar o header da biblioteca (proc_grafico.h) e após isso você pode chamar as funções criadas que citamos anteriormente. 

### inicializa_fpga e fecha_dev_mem
Antes de usar qualquer uma das funções gráficas da biblioteca você deve chamar o inicializa_fpga e ao encerrar a função gráfica fecha_dev_men para que as funções gráficas funcionem corretamente chamando as syscall necessárias para abrir e fechar o arquivo devmam, para fazer e desfazer o mapeamento. 

### escreve_bloco e apaga_bloco
Para usar a função escreve_bloco você precisa passar a cor e posição. Em relação as cores, elas já estam definidas no header da biblioteca e basta chamar o nome da cor, e em relação a posição é necessário fazer um calculo para passar o valor. Sabendo que em tela temos uma resolução de 80x60 blocos de tamanho 8x8, quando formos passar a posição precisamos calcular dessa maneira: (i + (j * 80)), em que i é a posição da linha e j a posição da coluna. Em relação ao apaga bloco, somente é preciso passar a posição em que quer apagar.
- Parâmetros
  - uint16_t cor: cor do bloco
  - uint16_t posicao: posicao do bloco
<div align="center">  
  <img align="center" width=90% src="img/escreve_bloco.png">
  <p><em>Chamada da função em C</em></p>
</div>

### altera_cor_bG
Após chamar essa função somente é necessário passar a cor e o registrador que irá armazenar a informação. 
- Parâmetros
  - uint16_t cor: o número da nova cor do background.
  - uint8_t registrador: o registrador onde a cor será armazenada.
<div align="center">  
  <img align="center" width=90% src="img/altera_cor_bg.png">
  <p><em>Chamada da função em C</em></p>
</div>

### exibe_sprite 
Para o exibe_sprite precisamos passar o sp como 1 para habilitar o sprit, o offset, o registrador e também a posição xy. Para passar a posição xy, fizemos uma mascara de bits para certificar que os bits que iriamos passar estavam precisos com o que queriamos passar, limitando os bits das posições x e y aos 10 bits menos significativos. Assim, a máscara de bits é usada nesse contexto para garantir que apenas os 10 bits necessários sejam usados ao combinar pos_x e pos_y em pos_xy_20b, que armazena os dois valores em 20 bits (10 bits de pos_x deslocados e 10 bits de pos_y).
- Parâmetros
  - uint8_t sp: habilita/desabilita sprite
  - uint16_t xy: posição x,y do sprite
  - uint16_t offset: deslocamento de acesso na memória de sprites
  - uint8_t registrador: registrador a ser utilizado
<div align="center">  
  <img align="center" width=100% src="img/exibe_sprite.png">
  <p><em>Chamada da função em C</em></p>
</div>

### desenha_poligono
Para essa precisamos passar a cor (que já foram definidas no header), o tamanho que varia entre 15 tamanhos diferentes (detalhados no TCC), a forma do poligono, em que 0 é quadrado e 1 é triângulo, e a posição xy em que para isso é necessário fazer o mesmo processo com máscara de bits que citamos anteriormente no exibe_sprit para passar exatamente, nesse caso, 18 bits referente a essas posições, sendo que 9 desses bits se refere ao x e 9 ao y. 
- Parâmetros
  - uint16_t cor: cor do poligono
  - uint16_t tamanho: qual seria o tamanho do poligono, podendo varias entre 16 tamanhos
  - uint16_t forma: define se é quadrado ou triângulo
  - uint16_t endereco: onde o polígono vai ser posicionado em tela, psoição xy
<div align="center">  
  <img align="center" width=100% src="img/desenha_poligono.png">
  <p><em>Chamada da função em C</em></p>
</div>

### altera_pixel_sprite
Aqui passamos a cor e o endereço do pixel que queremos que seja alterado com essa cor. 
- Parâmetros
  - uint16_t cor: cor do pixel
  - uint16_t endereco: endereço do pixel do sprite
<div align="center">  
  <img align="center" width=100% src="img/altera_pixel_sprite.png">
  <p><em>Chamada da função em C</em></p>
</div>

# Testes
Foram construidos 5 casos de testes para testar se todas instruções estam funcionando corretamente:

 - **Caso de Teste 1 | Instrução DP:** 
    Esse primeiro caso de teste tem por objetivo testar a função feita da nossa biblioteca: desenha_poligono, usando a intrução DP da GPU. Assim ela deve desenhar um poligono em tela, seja um triângulo ou quadrado.  

 - **Caso de Teste 2 | Instrução WBR para alterar o backgraund:** 
    Tem por objetivo testar a função altera_cor_bg da nossa biblioteca para alterar a cor do Backgraund. Nela usamos a intrução WBR da GPU. 

 - **Caso de Teste 3 | Instrução WBR para exibir sprite:** 
    Esse caso de teste tem por objetivo exibir um sprit em tela com a nossa função exibe_sprite, usando a intrução WRB da GPU. 

 - **Caso de Teste 4 | Instrução WSM para armazena ou modifica o conteúdo presente na Memória de Sprite:** 
    Tem por objetivo modificar o conteúdo na memória do sprite usando a instrução WSM e posteriormente exibir em tela para certificar a modificação. Nessa caso, desenhamos um pinguim em um sprit e exibimos essa modificação em tela. 

  - **Caso de Teste 5 | Instrução WBM configurar valores RGB para o preenchimento de áreas do background:** 
    Esse caso de teste tem por objetivo modificar uma área especifica do backgraund usando a instrução WBM. 
    Assim, com essa função conseguimos fazer todo nosso jogo tetris que iremos exibir para mostrar o efeitivo funcionamento usando as seguintes funções da nossa biblioteca: escreve_bloco e apaga_bloco.


# Tecnologias e Ferramentas utilizadas
- **Hardwares:**   
  - Kit de Desenvolvimento DE1-SoC
  - Monitor   
- **Linguagens:** Assembly e C
- **Ambiente de Desenvolvimento:** Visual Studio Code   
- **Compilador:** GCC  
- **Controle de Versão:** Git     
- **Ferramenta de Sistema:** Terminal Linux (Ubuntu)


# Configurações de Ambiente e Execução
Para ter acesso ao projeto, clone o repositório disponível na plataforma GitHub utilizando o seguinte comando no terminal Linux:
```bash
git clone https://github.com//sarinhasf/Biblioteca-GPU.git
```
Após clonar o repositório, conecte-se à placa via SSH utilizando o seu respectivo IP. Por exemplo, se o IP for `10.0.0.120`, use o seguinte comando:
```bash
ssh aluno@10.0.0.120
```
Em seguida, transfira a pasta clonada do seu computador para o sistema de arquivos da placa:
```bash
mv Biblioteca-GPU/[caminho do destino]
```
Para compilar e executar o projeto desenvolvido, navegue até o diretório onde está o repositório e execute o comando:
```bash
make
```
O comando `make` gerará o arquivo de compilação e o executará. Se a operação for bem-sucedida, a tela inicial do Candi Block deverá aparecer no monitor ao qual a placa está conectada.
<br>
⚠️ **Observação:** para seguir esse passo a passo será necessário saber a senha do usuário `aluno`.


# Desenvolvedoras
<table>
  <tr>
    <td align="center"><img style="" src="https://avatars.githubusercontent.com/u/142849685?v=4" width="100px;" alt=""/><br /><sub><b> Brenda Araújo </b></sub></a><br />👨‍💻</a></td>
    <td align="center"><img style="" src="https://avatars.githubusercontent.com/u/89545660?v=4" width="100px;" alt=""/><br /><sub><b> Naylane Ribeiro </b></sub></a><br />👨‍💻</a></td>
    <td align="center"><img style="" src="https://avatars.githubusercontent.com/u/143294885?v=4" width="100px;" alt=""/><br /><sub><b> Sara Souza </b></sub></a><br />👨‍💻</a></td>    
  </tr>
</table>


# Referências
- [1] FPGAcademy. (2024) https://fpgacademy.org/
- [2] Trabalho de Conclusão de Curso de Gabriel Sá Barreto Alves. (2024) https://drive.google.com/file/d/1MlIlpB9TSnoPGEMkocr36EH9-CFz8psO/view
  
<img width=100% src="https://capsule-render.vercel.app/api?type=waving&color=6959CD&height=120&section=footer"/>
