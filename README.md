# Desenvolva-um-Gerenciador-de-Pedidos-de-um-Restaurante-com-JavaScript-e-Kafka

Para desenvolver um Gerenciador de Pedidos de um Restaurante utilizando JavaScript e Kafka, precisaremos estruturar o projeto em microsserviços que se comunicam através de mensagens assíncronas utilizando o Kafka como broker de mensagens. Vou detalhar os passos e exemplos básicos para cada parte do projeto:

### Estrutura do Projeto

1. **Microsserviços:**
   - Cada funcionalidade do sistema (gerenciamento de pedidos, gestão de estoque, notificações, etc.) será um microsserviço separado.
   - Utilize Node.js para implementar cada serviço.
   - Cada serviço deve ser independente e se comunicar com os outros serviços através de mensagens Kafka.

2. **Kafka:**
   - Kafka será utilizado como um sistema de mensageria para enviar eventos entre os serviços.
   - Você precisará configurar um cluster Kafka localmente ou usar um serviço gerenciado (por exemplo, Confluent Cloud, AWS MSK, etc.).

3. **Gerenciamento de Pedidos:**
   - Um dos microsserviços será responsável por gerenciar os pedidos, recebendo requisições para criar novos pedidos, atualizar status, etc.

### Exemplo de Implementação

Vamos exemplificar a implementação de um serviço simples de Gerenciamento de Pedidos em Node.js usando Kafka para comunicação assíncrona entre serviços.

#### Estrutura de Diretórios

```
├── gerenciamento-pedidos-service
│   ├── package.json
│   ├── index.js
│   ├── kafka.js
│   ├── pedidos-controller.js
│   ├── pedidos-service.js
│   └── ...
└── kafka-config.js
```

#### Configuração do Kafka (`kafka-config.js`)

```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
  clientId: 'meu-app-client',
  brokers: ['localhost:9092']  // Endereço do broker Kafka
});

const producer = kafka.producer();

module.exports = { kafka, producer };
```

#### Pedidos Controller (`pedidos-controller.js`)

```javascript
const { producer } = require('./kafka-config');

async function criarPedido(req, res) {
  try {
    const { cliente, itens } = req.body;

    // Validação e criação do pedido

    await producer.send({
      topic: 'novo-pedido',
      messages: [
        { value: JSON.stringify({ cliente, itens }) }
      ]
    });

    res.status(201).json({ message: 'Pedido criado com sucesso.' });
  } catch (error) {
    console.error('Erro ao criar pedido:', error);
    res.status(500).json({ error: 'Erro interno ao criar pedido.' });
  }
}

module.exports = { criarPedido };
```

#### Pedidos Service (`pedidos-service.js`)

```javascript
const { kafka } = require('./kafka-config');

async function processarNovoPedido(message) {
  const { cliente, itens } = JSON.parse(message.value.toString());

  // Lógica para processar o novo pedido

  console.log(`Novo pedido recebido: Cliente ${cliente}, Itens: ${JSON.stringify(itens)}`);
}

const consumer = kafka.consumer({ groupId: 'pedidos-group' });

async function run() {
  await consumer.connect();
  await consumer.subscribe({ topic: 'novo-pedido' });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      await processarNovoPedido(message);
    },
  });
}

run().catch(console.error);

module.exports = { processarNovoPedido };
```

#### Index.js (Ponto de Entrada)

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const { criarPedido } = require('./pedidos-controller');

const app = express();

app.use(bodyParser.json());

app.post('/pedidos', criarPedido);

const port = 3000;
app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});
```

### Considerações Finais

Este é um exemplo básico para demonstrar como podemos estruturar um serviço de gerenciamento de pedidos utilizando Kafka e Node.js. Para um sistema real, precisaremos expandir essa estrutura, implementar a lógica de negócios adequada, garantir a escalabilidade e a resiliência dos serviços, entre outros aspectos importantes. Além disso, considere utilizar ferramentas de monitoramento e gerenciamento de microsserviços para facilitar a operação do sistema em produção.
