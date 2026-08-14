# Caderno_notas&aprendizados

Projeto Korp — Sistema de Emissão de Notas Fiscais
-  Este documento registra os principais conceitos, decisões e aprendizados adquiridos durante o desenvolvimento do projeto. O objetivo não é apenas documentar o que foi feito, mas registrar o raciocínio por trás da solução.

  
# 1. Entendimento do Projeto.
O desafio consiste no desenvolvimento de um sistema de emissão de notas fiscais, utilizando Angular no frontend e uma arquitetura com, no mínimo, dois microsserviços: 
- Serviço de Estoque - responsável pelo controle de produtos e saldos; 
- Serviço de Faturamento - responsável pela gestão das notas fiscais.

Também é exigida a persistência dos dados em um banco de dados real e a implementação de um cenário em que um dos microsserviços falhe, com tratamento e feedback apropriado ao usuário. O banco de dados é de escolha do candidato. Neste projeto, optei pelo PostgreSQL.


# 2. Visão Geral da Arquitetura.
A primeira compreensão sobre o desenvolvimento foi seperar as responsabilidades e dissecar os caminhos:

USUÁRIO -> ANGULAR -> API/HTTP -> BACKEND -> BANCO DE DADOS

Para dissecar o caminho, primeiro precisei entender sobre como cada tecnologia deve interagir: Angular pergunta e apresenta. A API transporta. O Backend processa. O Banco armazena. Simples de entender, mas de extrema importância.

# 3. Microsserviços.
O desafio exige dois microsserviços: Estoque e Faturamento.

A ideia é separar responsabilidades.

- Estoque

Qual produto existe?
Qual é o saldo?
Posso retirar determinada quantidade?
Qual será o novo saldo?

- Faturamento

Qual nota existe?
Qual é o número da nota?
Qual é o status?
Quais produtos fazem parte da nota?
A nota pode ser impressa?

- Decisão

Não será criado um terceiro microsserviço apenas para aumentar a complexidade, tendo em vista que estou trabalhando com novas tecnologias e o foco deve permanecer na qualidade e organização.


ANALOGIAS: 

Identificadores e PRIMARY KEY; id SERIAL PRIMARY KEY - Podem existir vários "Joãos" e "Marias". O nome, sozinho, não necessariamente identifica uma única pessoa. A chave primária funciona como um identificador único dentro daquela tabela.

Relação entre Nota e Produto; Uma nota pode conter vários produtos, porque "João" e "Maria" podem comprar vários produtos. Se ambos forem consumistas, é necessário prestar atenção a relação estabelecida entre a Nota -> Produto e Produto -> Estoque.


# 4. Concorrência.
Uso opcional, mas de extrema necessidade. O sistema não pode permitir que as duas operações retirem a mesma unidade de forma inconsistente.
A concorrência é tecnicamente interessante, mas será considerada somente depois que todos os requisitos obrigatórios estiverem funcionando.

# 5. RxJS e Processos.
Biblioteca utilizada para trabalhar com fluxos de informações que acontecem ao longo do tempo. No Angular, isso aparece principalmente na comunicação assíncrona. Sobre isso, tentei pensar com uma analogia - IFOOD:

  Angular
  "Quero um combo do BK!"
      
  API/HTTP 🛵
  Leva o pedido
      
  Backend
  Prepara a resposta
      
  API/HTTP 🛵
  Traz a resposta
      
  Observable 👀
  Acompanha o resultado
      
  RxJS 🧰
  Trabalha com o resultado
      
  Angular
  Mostra na tela

Mas por que essas adições? Bem, durante o estudo do Angular, surgiu a necessidade de entender como o sistema lida com informações que não chegam imediatamente. Por exemplo, quando o Angular solicita os produtos ao Backend, o backend precisa processar a solicitação e depois devolver uma resposta. O Angular não sabe exatamente quando essa resposta chegará. Foi nesse contexto que conheci o Observable.

Podemos imaginar o seguinte fluxo: Eu faço um pedido no iFood.

O pedido passa por diferentes etapas: Pedido realizado -> pedido confirmado -> preparando -> a caminho -> entregue.

O sistema precisa acompanhar algo que está acontecendo ao longo do tempo, até que exista uma resposta ou resultado. No Angular, algo semelhante acontece: Angular faz uma solicitação -> backend processa -> resposta fica disponível -> Angular recebe a resposta -> Angular reage.

O Observable representa esse fluxo de informações que pode produzir resultados ao longo do tempo. E onde entra o RxJS? O RxJS é uma biblioteca que fornece ferramentas para trabalhar com esses fluxos.

Mas se o Angular vai receber a resposta de qualquer forma, que diferença faz o Observable? Voltamos ao IFOOD! Após fazer um pedido, você não precisa ficar parado olhando para o restaurante, ou aplicativo. Você pode continuar usando o celular e quando acontecer alguma coisa, o aplicativo te avisa. O Observable é parecido com isso.

E o RxJS? Ele é quase como: "Quando chegar a resposta, faça isso". Mas por que não simplesmente esperar a resposta? Porque enquanto o backend está trabalhando, o sistema pode continuar funcionando. Então o Observable o o RxJS são como partes do mesmo corpo, um é os olhos, o outro, os braços.

# 6. "Perguntas que o projeto me fez".
- Onde cada dado será armazenado?
  Produtos, código, descrição e saldo: pertencem ao Serviço de Estoque e serão persistidos no PostgreSQL.
  Notas, número, status e itens da nota: pertencem ao Serviço de Faturamento e serão persistidos no PostgreSQL.
  A estrutura exata das tabelas ainda será definida.

- Quem pode alterar esse dado?
  Produtos e saldo: Serviço de Estoque.
  Notas e status: Serviço de Faturamento.
  O Angular não altera diretamente o banco; ele solicita operações através da API.
  
- Quem precisa consultar esse dado?
  O Angular precisa consultar produtos e notas para apresentar as informações ao usuário.
  O Faturamento precisa consultar informações do Estoque durante a emissão da nota.
  O Estoque precisa receber a solicitação de atualização do saldo.

- Como um serviço conversa com o outro?
  Por meio de API/HTTP.
  
- O que acontece se um serviço estiver indisponível?
  O sistema deve identificar a falha e informar o usuário de maneira apropriada, conforme requisito obrigatório do teste.
  
- O que acontece se o usuário repetir uma ação?
A nota começa como Aberta -> é emitida -> estoque é atualizado -> nota passa para Fechada -> uma nota fechada não pode ser impressa novamente.
Isso já é uma proteção prevista no requisito obrigatório, estão descartei a idempotência.

# Estas são decisões iniciais. Elas serão refinadas durante a modelagem do banco e a implementação dos microsserviços.
  
