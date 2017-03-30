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
digital. Nós o chamaremos de simple.py.

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
    #Aqui instanciamos o
    dut = circuito_a_ser_testado(signals)
    #Esse método gera os sinais de entrada no circuito pro teste
    f_estimulo = estimulo(signals)
    # Esse método verifica as saídas do circuito para garantir a corretude.
    f_verifica = verificacao(signals)
    return dut, f_estimulo, f_verifica
```

Cada uma das funções listadas no return representam para o MyHDL um circuito
digital. Estruturando a função bench desse modo podemos reutilizá-la
independente de que circuito estamos testando. Isso vai acelerar a escrita de
novos testes.

E como seria um teste?

``` python

def test_aquele_um_porcento():
    '''
        Teste que verifica....
    '''
    def estimulo():
        '''
            Função que cria os estímulos para o caso de teste
        '''
        pass

    def verificacao():
        '''
            Função que verifica as saídas e no geral controla a duração da
            simulação
        '''
        pass

    # Criamos o objeto responsável por executar a simulação
    simulator = myhdl.Simulation(bench(estimulo, verificacao))
    # executamos a simulação
    simulator.run()

```

Nós iremos preencher as lacunas e definir como se dá a interação entre
cada um dos elementos da nossa simulação nas próximas seções. Um ponto
importante a destacar:

**Cada uma das funções declaradas modela um circuito digital independente e
deve ser implementada considerando que sua execução é paralela**

## Descrição dos circuitos

O primeiro ponto a compreender é que estamos desenvolvendo um circuito. No fim
do nosso projeto teremos algo que irá se relacionar com o mundo através de um
conjunto de interfaces elétricas. Então precisamos deixar claro quais são os
"pinos" do nosso circuito.

Vamos dar uma olhada no arquivo simple.py.

``` python
from collections import namedtuple
import myhdl as hdl

Parameters = namedtuple('Parameters', 'nleds')

class CircuitPorts(object):
    '''
        This is the place where we declare the circuit ports
    '''
    def __init__(self):
        pass

def circuit(self, signals):
    pass

```

Temos declarados três componentes importantes no código acima:

    - Parameters: Essa classe é uma namedtuple que irá descrever os parâmetros
      variáveis do nosso circuito. Ou seja se quisermos criar variações do
      circuito serão esses os parâmetros alteráveis.
    - CircuitPorts: Essa classe determina quais são as portas do nosso circuito
      digital.
    - circuit: Essa função contém a descrição do nosso circuito digital. E ela
      que será chamadad quando quisermos utilizar o circuito.

# Nosso circuito inicial

Para ilustrar o nosso fluxo antes de partirmos para o projeto que nos propusemos
a fazer iremos criar um contador que irá se ligar a um display de sete
segmentos.

## Comportamento do circuito principal

Vamos inicialmente tratar do nosso circuito principal. Esperamos que o seguinte
aconteça:
- Nosso contador se iniciará em zero.
- Ao apertarmos um botão o dispositivo acrescentará 1 a contagem.
- Um segundo botão reduzirá a contagem de 1.

### O teste do nosso circuito principal

Para testar o nosso projeto vamos criar na pasta test o arquivo test_design.py.

Com o seguinte conteúdo:

``` python
import myhdl as mhd
import

def bench(stimulus, verification):
    pass

def test_initial_state():
    '''
        Test that the initial behavior is correct.
        - The seven segment display must show zero
    '''
    def stimulus():
        pass

    def verification():
        pass

    simulator = mhd.Simulation(bench(stimulus, verification))
    simulator.run()

```

Observe que estamos revisitando a estrutura que usamos de exemplo para o teste.
Vamos focar agora na nossa função bench.

O papel principal da função bench e servir de estrtutura para o teste do nosso
circuito evitando que tenhamos a reescrita de uma estrutura muito reutilizada.
Nós poderíamos mover essa função para um pacote e reutilizar em todos os testes
aumentando assim a reutilização dessa infraestrutura e a reutilização de código.
Por ora vamos voltar a criação do nosso teste.

O objetivo da função bench é prover um circuito digital a ser utilizado pros
testes

``` python
def bench(stimulus, verification, circuit_parameters):
    signals = design.CircuitPorts(circuit_parameters)
    dut = design.circuit(signals)
    f_estimulo = stimulus(signals)
    f_verifica = verification(signals)
    return dut, f_estimulo, f_verifica
```

No exemplo acima temos a descrição do circuito digital. Em um diagrama temos
algo assim:

### Inserir imagem do teste

Vamos agora olhar a implementação do teste:

``` python
def test_initial_state():
    '''
        É importante deixarmos claro qual o objetivo do teste. Isso nos ajuda a
        recordar ou descobrir o que motivou as escolhas do desenvolvedor no
        passado.

        Esse primeiro teste valida que no início o display mostrará o valor
        zero.
    '''

    # Nessa declaração inicial temos os parâmetros do circuito sob teste
    circuit_param = design.Parameters(nleds=8)

    def stimulus(signals):
        '''
            Dado que o processo de inicialização é interno não precisamos de
            estímulo ao circuito digital nesse ponto.abs
        '''
        yield mhd.delay(1)

    def verification(signals):
        yield mhd.delay(1)

    simulator = mhd.Simulation(bench(stimulus, verification, circuit_param))
    simulator.run(10)
```

É importante destacar as ideias contidas nessa estrutura. Primeiro a função
bench que funciona como um diagrama de circuito entre CIs, você pode considerar
como um esquemático. Escrever HDL nesse estilo é conhecido como estrutural.

Salvo o uso da sintaxe do Python e nossos esforços para deixar a coisa mais
legível e reutilizável, o conceito é o mesmo encontrado no VHDL e Verilog.

Repare que nesse ponto não inserimos ainda um teste propriamente dito, mas o
código acima já funciona e nos diz que o que implementamos está correto.

Podemos validar a implementação acima executando:

```
python -m pytest
```

Na execução do teste obtemos a resposta que 1 teste foi executado e passou.

Observem que inserimos o yield nas funções de teste e executamos o método run
com o valor 10.

Precisamos inserir o yield para devolvermos o controle para a simulação para que
ela decida o próximo passo. Funciona da seguinte maneira, todos os módulos
executam em pararlelo. Para emular esse cenário o estado da simulação é alterado
por um bloco de cada vez e eles são sincronizados ao final de cada ciclo, com
isso cada bloco opera com os valores do final do último ciclo e atualiza os
sinais para o próximo ciclo. O parâmetro que inserimos no método run serve para
controlarmos o número de ciclos de simulação. Isso não será necessário quando
inserirmos a verificação através de assert na função verification.

Para esse teste a nossa função stimulus fica exatamente daquele modo. Queremos
validar que ao ligar o dispositivo a informação correta estará no display. Para
outros circuitos essa função fica naturalmente mais complexa. Teremos exemplos
desse tipo na sequência dessa série.

## Implementando a função verification

No nosso teste queremos validar que ao inicializar o circuito envia para a saída
leds os sinais que farão o display de sete segmentos mostrar o valor 0. Para
isso usaremos o assert do python. A função fica assim:

```python
def verification(signals):
    assert signals.leds == 0, 'The display must show Zero'
    yield mhd.delay(1)
```

Se executarmos novamente o teste veremos que ele irá passar. É também uma boa
prática forçar que o seu teste falhe pra você garantir que o seu teste está
sendo executado que ele realmente detectará a falha que você pretende evitar com
o teste.

Se você está um pouco mais atento a leitura irá detectar que o nosso teste
possui um bug! Ele foi inserido ali propositalmente para alertar que o teste
automatizado é uma ferramenta importantíssima, mas não nos priva de cometermos
erros no próprio teste. Por isso é importante que o teste escrito seja tão
isolado quando possível para tornar simples a detecção das falhas que venham a
acontecer no próprio teste.

O código que temos até o momento pode ser encontrado [aqui](https://github.com/euripedesrocha/simple_hdl/tree/test_main_circuit).

# Próximo artigo

Na sequência dessa série vamos corrigir o teste e estabelecer uma arquitetura
particionando os blocos do nosso sistema, escrever circuitos combinacionais e
sequenciais e escrever um gerador de clock para os nossos testes.
