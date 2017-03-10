# Intro

Olá, nos últimos tempos tenho observado um crescente interesse por projetos
utilizando FPGAs, isso se dá talvez por uma certa maturidade atingida pelas
tecnologias e o aumento da disponibilidade de opções.

Entretanto esse interesse vem seguido tipicamente de um frustração em virtude da
complexidade típica encontrada na inicialização de um projeto utilizando FPGA,
assunto já abordado pelo Francesco [aqui](fix_link_artigo.embarcados.com.br).

Por isso resolvi assumir a tarefa de executar um projeto um pouco mais complexo
desde a sua concepção construindo todas as etapas e tentando compartilhar um
pouco da maneira de como eu penso deve seguir um projeto com esse tipo de
dispositivo.

## O que faremos?

Uma aplicação interessante para FPGAs é a construção de aceleradores para
funções de alta carga de processamento matemático. FPGAs entregam resultados em
um tempo constante e algumas vezes consumirão menos energia que um processador
ou GPU. Como não pretendo nos levar por um projeto de grande porte vamos pensar
em uma aplicação de processamento digital de sinais de áudio e vamos construir
um efeito para guitarra elétrica. Adiante discutimos qual efeito e que tal me
ajudar a escolher o que iremos implementar deixando o seu voto nos comentários?

## Que FPGA vamos utilizar?

Inicialmente não usaremos nenhum. As opções são muitas e na realidade pra
estruturação inicial do nosso material isso não é necessário.

Mesmo sem uma escolha inicial devemos estabelecer uma lista inicial baseada nos
requisitos que devemos atender.

- Preço.
    O primeiro objetivo é que o nosso design possa competir em custo de
    hardware com um microcontrolador de médio desempenho. Vamos atribuir um
    preço unitário na casa de US$ 10,00 no máximo
- Presença de hardware para multiplicação.
    Embora os elementos internos do FPGA sejam reconfiguráveis é interessante
    atentar para a presença de blocos especializados para a nossa aplicação. No
    caso construiremos uma aplicação que aplica algoritmos de processamento
    digital de sinais então é bom que tenhamos multiplicadores disponíveis.

# Sequência de trabalho

Vamos ser sinceros, projetos com lógica programável geralmente atrasam e possuem
bastante esforço na correção de bugs e retrabalho. Infelizmente essa é a
realidade da indústria hoje. FPGAs são dispositivos tidos como complexos de
trabalhar. Isso não é mentira, mas microcontroladores ou mesmo uma placa com
arduino não são muito mais simples. O fato é que nos projetos microprocessados
nos fazemos de uso de ferramentas e de um conjunto razoável de código escrito
previamente.

Quando se trata de projetos com FPGA isso não é verdade. Muitas vezes o
projetista acaba tendo de escrever seu projeto do zero e a reutilização de
módulos prontos traz consigo a complexidade da integração.

Nessa sequência de textos nós vamos tentar resolver esses problemas com duas
abordagens:

- Verificação automatizada e com modelos escritos em alto nível
- Construção de uma biblioteca de circuitos digitais, pensada desde o princípio
  para se integrar.

É um desafio de tamanho razoável mas eu acredito que conseguiremos vencê-lo!

# Ferramentas

Já que decidimos postergar a escolha do dispositivo as ferramentas para as
etapas que geram o circuito para funcionar em um FPGA serão apresentadas
posteriormente.

Precisamos primeiro decidir que HDL usar. Você nesse momento deve estar se
perguntando o que é HDL? HDL significa Hardware Description Language, em bom
português Linguagem de Descrição de Hardware. São essas linguagens que nos
permitem descrever textualmente circuitos de lógica digital.

As principais linguagens desse tipo são:

- VHDL
- Verilog

Recentemente além dessas o SystemVerilog vem criando força na indústria.

A despeito das linguagens citadas serem as mais comuns nós iremos utilizar uma
outra HDL que já é conhecida do leitor do embarcados: MyHDL.

Essa escolha se dá por alguns motivos principais:

- O uso do python possibilita a construção de um ambiente de verificação de
  circuitos muito poderoso
- MyHDL possui um modelo de descrição de hardware que está no mesmo patamar do
  VHDL e Verilog, aliado a sintaxe limpa do python isso o torna uma excelente
  HDL inicial
- Entregando o mesmo tipo de abstrações MyHDL consegue gerar código Verilog e
  VHDL inteligível, sem que pareça algo gerado automaticamente
- Com uso do MyHDL podemos utilizá-lo em outros cenários para a verificação do
  Verilog e VHDL acrescentando assim uma linguagem de verificação muito
  poderosa ao conjunto de ferramentas do desenvolvedor.

# A estrutura do nosso repositório de trabalho

Vamos começar com uma estrutura de exemplo pra mostrar como será o nosso fluxo
de trabalho.

Pra estruturar nosso trabalho vamos nos valer de três pilares:

- Desenvolvimento
- Testes
- Síntese e geração da imagem do FPGA

Nessa etapa inicial vamos focar em poucos módulos pra facilitar a compreensão de
cada um deles.

Nosso primeiro arquivo se chama build.py. Nele nós vamos montar um aplicativo de
linha de comando pra executar as etapas de teste e geração da imagem.

O outro arquivo que iremos tratar será o arquivo principal do nosso circuito
digital. Nós o chamaremos de top_circuit.py.

Entraremos em detalhes sobre cada um deles mais adiante.

Como estamos descrevendo nosso circuito digital usando python a estrutura será a
mesma de um pacote python, podemos inclusive entregar o nosso circuito via
Python Package Index, e usando um simples "pip install" um possível usuário ter
disponível o circuito.

Além desses nós vamos lidar com um conjunto de testes. O desenvolvedor de HDL
mais experiente vai certamente identificar a estrutura de teste e a criação do
testbench e espero vai concordar comigo que escrever os testes usando uma
linguagem como Python proporciona um conjunto de testes mais poderoso.

## Estrutura de testes

Para estruturar nossos testes vamos usar a biblioteca py.test. Todos os nossos
testes estarão da pasta chamada test e deverão possuir o prefixo test\_.

Para cada teste iremos acrescentar uma função chamada bench (que pode ser
traduzido como bancada, logo test bench é a nossa bancada de testes). Essa
função é responsável por fazer a ligação entre os sinais de entrada e a saída do
circuito que iremos validar. Com isso é bom que nossos testes possuam uma
granularidade boa em termos de circuito, no software esses testes são conhecidos
como testes unitários. 

Nossa função bench terá a seguinte estrutura:

``` python
def bench(estimulo, verificacao, test_parameters):
    # Aqui colocamos os sinais necessários ao teste
    signals = SignalList()
    # Aqui instanciamos o 
    dut = circuito_a_ser_testado(signals)
    # Esse método gera os sinais de entrada no circuito pro teste
    f_estimulo = estimulo(signals)
    # Esse método verifica as saídas do circuito para garantir a corretude.
    f_verifica = verificacao(signals)
    return dut, f_estimulo, f_verifica
```

Cada uma das funções listadas no return representam para o MyHDL um circuito
digital. Estruturando a função bench desse modo podemos reutilizá-la
independente de que circuito estamos testando. Isso vai acelerar a escrita de
novos testes.

## Descrição dos circuitos

O primeiro ponto a compreender é que estamos desenvolvendo um circuito. No fim
do nosso projeto teremos algo que irá se relacionar com o mundo através de um
conjunto de interfaces elétricas. Então precisamos deixar claro quais são os
"pinos" do nosso circuito.

# Nosso circuito inicial

## Um teste para um contador
## Instanciando o contador no circuito principal
## Comportamento do circuito principal
## Descrevendo um contador
## Gerando o vhdl/verilog

# Próximo artigo
