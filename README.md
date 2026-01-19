# Sistema de venda de ingressos com alta disponiblidade

Arquitetura proposta para um desafio da empresa **BigDataCorp**, focada em escalabilidade massiva, consistência de dados e justiça no processamento de pedidos.

---

## O Desafio
Imagine que você é o(a) arquiteto(a) responsável pelo desenho de um sistema de logins para um site extremamente concorrido de ingressos de um mega show de Rock em Rio (got it?). Dado que o número de ingressos é limitado e muito inferior a quantidade de acessos no dia venda, você precisa garantir que o site só irá finalizar a venda para pessoas que realmente vão receber o ingresso, ou seja, você não pode deixar uma pessoa comprar um ingresso sem que haja mais disponíveis. Além disso, um cliente com internet mais lenta não ficaria feliz de não conseguir comprar seu ingresso pois uma pessoa com internet mais veloz passou sua frente.

Desenhe uma estrutura de software que sustente esse serviço, pode ser um desenho simples, da maneira que você preferir, desde que o mesmo transmita a ideia da arquitetura que você teve para quem o lê. Colocamos aqui um exemplo de desenho de uma arquitetura aleatória para servir de inspiração.

http://help.market.com.br/csharp/hmfile_hash_6c8b0b18.png

Para nos enviar seu desenho (resposta desse exercício), suba a imagem do mesmo no seu GitHub em um repositório publico e compartilhe o link aqui na caixa de respostas.

Boa sorte!

---

## 🛠️ Stack Tecnológica (AWS)

* **Front-end:** React hospedado com **CloudFront** (CDN).
* **Segurança:** **AWS WAF** para bloqueio de bots e proteção contra DDoS.
* **API Layer:** **API Gateway** para gerenciamento de requisições http.
* **Fila de Espera:** **AWS SQS** para garantir a ordem de chegada e desacoplar o tráfego.
* **Check de Estoque:** **Amazon Dynamo** para checagem de estoque.
* **Processamento:** **AWS Lambda** (Workers assíncronos).
* **Banco de Dados:** **Amazon DynamoDB** (NoSQL com transações ACID).
* **Orquestração:** **AWS Step Functions** para gerenciar o ciclo de vida do pagamento e reserva.
* **Notificações:** **Amazon SNS** para confirmação de compra em tempo real.

---

## 🚀 Diferenciais da Solução 

1.  **Justiça no Acesso:** O uso de filas **SQS** garante que o processamento respeite a ordem de chegada, neutralizando a vantagem de usuários com internet de ultra velocidade.
2.  **Prevenção de Overselling:** Implementado um fluxo de "Reserva Temporária" (Lock) no DynamoDB com TTL, garantindo que um ingresso não seja vendido duas vezes.
3.  **Resiliência:** A arquitetura é totalmente desacoplada. Se o gateway de pagamento oscilar, a mensagem permanece na fila para retry automático sem perda de dados.
4.  **Performance:** O check de "Ingressos Esgotados" acontece no Redis, evitando consultas desnecessárias ao banco de dados principal durante o pico.

---

## 📈 Fluxo do Usuário
1. Usuário acessa via CloudFront.
2. Requisição de compra cai no SQS.
3. Worker processa a fila e valida estoque no Dynamo.
4. Step Functions coordena o pagamento externo.
5. Se aprovado, o status muda para `VENDIDO` e o SNS dispara o voucher.