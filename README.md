# Sumário
* [O Problema](#o-problema)
* [A Solução](#a-solução)
* [O Protótipo](#o-protótipo)
    * [Principais Desafios](#principais-desafios)
    * [Arquitetura](#arquitetura)
* [Validação e Resultados](#validação-e-resultados)
* [Funcionamento](#funcionamento)
* [Conclusão](#conclusão)

# O que é

 O projeto apresenta uma balança inteligente de baixo custo que utiliza visão computacional para otimizar os sistemas de caixas de autoatendimento atuais, automatizando a identificação e pesagem de Frutas e Vegetais.

# O Problema

 Os caixas de autoatendimento foram sem dúvidas uma das maiores revoluções para o mundo dos supermercados. Sua função é agilizar o processo de pesagem e pagamento de produtos, além de reduzir a necessidade de interferência humana.
 Porém, existem situações em que essa tecnologia, criada para promover agilidade e autonomia, pode acabar provocando o efeito oposto.
### Situação
 Imagine-se numa situação em que você está com pressa, dando uma visita rápida ao mercado apenas para comprar o necessário. Você corre pelos corredores, colocando no carrinho todos os itens que precisa e torcendo para não pegar fila no pagamento.
 
 Mas o seu medo se prova real.
 
 Você vê filas enormes frente à todas as esteiras, instantâneamente se convencendo de que vai chegar atrasado no seu compromisso. Você então respira fundo e observa o lugar com mais atenção, percebendo alguns caixas de autoatendimento vazios. Com esperança brilhando em seus olhos, você vai até eles e começa a passar suas compras, agradecendo pelo milagre da tecnologia. Até você colocar as maçãs de seu carrinho na balança.

 A máquina não responde.
 
 Seus olhos passam a buscar por um antendente para te ajudar, mas havia apenas um ali. Ele atendia um homem que parecia estar levando o Hortifruti inteiro para casa. Por um instante, você sentiu raiva por ter apenas uma pessoa trabalhando ali, até pensou em reclamar, mas sentiu pena do funcionário ao vê-lo ter de digitar os códigos de cada fruta, uma por uma, no caixa. Realmente não havia uma forma mais fácil de fazer isso?
 Bem, é justamente isso que a nossa solução oferece.

# A solução

 Nossa ideia é unir a balança e uma câmera integrada com um modelo de inteligência artificial. Usando a IA para automatizar o processo de identificação e pesagem de frutas e vegetais. O sistema funciona da seguinte forma:
 - Cliente posiciona o item na balança
 - Câmera captura diversas imagens do item e envia para a IA
 - IA identifica o objeto e retorna seu nome e valor
 - A balança pesa o item
 - O sistema então calcula o preço total a ser pago e o exibe

 Parece simples de mais, não é?
 
 Talvez bom até de mais para ser verdade. 
 
 Mas fique tranquilo. Abaixo mostrarei o protótipo funcional, junto com os dados que pudemos adquirir em nossos testes. Ressaltarei também as virtudes e problemas envolvidos no projeto.
 
# O Protótipo

 Usamos um micro-controlador Arduino junto a um módulo de carga de 10kg para realizarmos a pesagem. Os dados coletados por eles eram enviados para um notebook que, além de realizar os cálculos necessários, também usava sua própria câmera para rodar a IA responsável pela identificação dos itens.
 
## Principais desafios

### Calibração da balança:

 Quando ligamos o protótipo pela primeira vez, tivemos a esperança de ver uma medição precisa e rápida no monitor.
 
 Pena que ficou só na esperança mesmo. 
 
 O sistema nos retornou números enormes que não pareciam fazer sentido algum. Então nós estudamos as razões do problema e como resolvê-lo, chegando a uma única resposta: **Calibração**.
 
 Nós então dividimos o problema em fatias e o resolvemos parte por parte.
 
 Primeiro lidamos com o "peso fantasma" da própria estrutura do protótipo (prato, parafusos, etc), que fazia a balança retornar valores mesmo estando vazia. Para fazer isso usamos este peso falso para determinar um ponto zero (variável **Offset**) 
 
 Em seguida colocamos pesos conhecidos na balança e anotamos os valores retornados por ela, usando os resultados para calcularmos nosso fator de escala (variável **Slope**).
 
 Após implementar estas simples, porém demoradas, soluções, nossos problemas estavam finalmente acabados!
 Ou pelo menos foi o que achamos...
 
 ### Filtragem da medida
 
 O valor apresentado pelo sistema estava finalmente condizendo com o do peso sobre a balança, só que tinha um problema: a leitura estava oscilando muito. A primeira vista aquilo não fez sentido algum. O peso estava estático, então por que a medição estava mudando?
 
 Foi nesse momento que percebemos que o mundo real é bem diferente de um software de simulação. As microvibrações da mesa, o ruído elétrico dos cabos soldados e até mesmo a temperatura do próprio circuito da célula de carga alteravam a medição, fazendo com que ela fosse imprecisa.
 
 Como quase tudo nessa vida, esse problema poderia facilmente ser resolvido com dinheiro, mas como os bons estudantes universitários que éramos, não tinhamos algum. Por isso optamos pelo plano B: 
 
 Matemática.
 
 Alteramos o sistema para funcionar da seguinte forma:
 
 Ao invés de ler o valor pós calibração e já mostrá-lo logo na tela, fizemos com que o sistema armazenasse suas 10 primeiras leituras e fizesse uma média com elas, repetindo este processo cinco vezes. Então organizamos os 5 valores em ordem e pegamos o do meio (mediana). Isso tirou a nossa medição da montanha russa que ela parecia estar presa, mas ainda havia um problema.
 
  Mesmo que muito menos que antes, a leitura ainda variava um pouco, como se agora estivesse num pequeno carrosel, subindo e descendo. Provavelmente não seria um problema grande no produto final, mas queríamos deixar a leitura mais fiel possível.
  
  Foi então que descobrimos o **Filtro EMA** (Média Móvel Exponencial), um método de filtragem que usa o valor das leituras passadas para determinar a atual, fazendo a medição "deslizar" dos valores mais distantes até o que queríamos.
  
  Após aplicarmos estas mudanças, obtivemos os seguintes resultados:
  
## Validação e Resultados

**Confiabilidade da balança:** Pelas limitações de hardware mencionadas anteriormente, ainda tivemos uma variação de 1 a 4 gramas do peso real dos items. Mas isso pode ser facilmente resolvido com hardware de melhor qualidade.

**Acurácia da IA:** A identificação sob iluminação ideal (>700 lux) atingiu uma precisão de 96,7%. Sua maior queda sendo com iluminação abaixo de 300 lux, caindo para 66,7% no pior caso.
## Arquitetura
<img width="421" height="125" alt="Diagrama do funcionamento da balanca" src="https://github.com/user-attachments/assets/0f21b2a5-634a-42df-9e9e-d49e10da71da" />

Como demonstrado no diagrama acima, o nosso protótipo faz uso dos seguintes componentes:

- Módulo de carga: Responsável pela pesagem dos itens
- Coversor AD HX711: Responsável por ampliar e converter os dados analógicos enviados pelo módulo de carga para sinais digitais, que são posteriormente enviados para o microcontrolador
- Arduino Mega: Microcontrolador que optamos por utilizar neste protótipo. É responsável por controlar a transição de sinais e dados entre os componentes, além de realizar operações matemáticas como a própria calibração da balança
- Computador: No protótipo, usamos um computador para rodar o script python responsável pela inicialização do modelo de IA, utilizando uma câmera ligada ao próprio computador para realizar a identificação das frutas e vegetais. O script também mostra na tela os objetos identificados, o peso detectado e o valor total da compra.

## Funcionamento

Segue abaixo um vídeo mostrando o funcionamento do protótipo.

https://github.com/user-attachments/assets/e969aca2-2922-4cfa-acbf-6861fbd4e168

## Conclusão

Este projeto foi desenvolvido ao longo de 6 meses como parte da disciplina de Projeto de Sistemas Embarcados no IBMEC-RJ.
Embora este README ofereça uma visão geral da jornada de desenvolvimento e de seus desafios, dados mais específicos como a metodologia científica, análises de precisão e os testes de acurácia da IA podem ser consultados no nosso artigo completo.
Agradeço pelo interesse em nosso projeto. Se tiver qualquer dúvida, sinta-se a vontade para abrir uma Issue
