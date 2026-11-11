# Frutaí

Site de pedidos e painéis do hortifrúti Frutaí (Ceasa, Teresina).

- `index.html`: site dos clientes (produtos, pedido com endereço de entrega e acompanhamento)
- `painel/index.html`: painel de pedidos, produtos e preços de venda, configurações da loja (login pelo Firebase)
- `financeiro/index.html`: painel financeiro (resumo, vendas do dia, compras e despesas, perdas, custos e taxas, precificação)

Os dados ficam no Firebase (projeto `frutai-c468e`).

## Regras do Firestore

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Catálogo e dados da loja: qualquer pessoa lê, só a equipe altera
    match /publico/{doc} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    // Pedidos: o cliente cria e acompanha o próprio pedido pelo código; só a equipe lista e altera
    match /pedidos/{id} {
      allow get: if true;
      allow create: if request.resource.data.status == 'novo'
                    && request.resource.data.origem == 'site'
                    && request.resource.data.criado == request.time
                    && request.resource.data.itens is list
                    && request.resource.data.itens.size() > 0
                    && request.resource.data.itens.size() <= 80
                    && request.resource.data.total is number;
      allow list, update, delete: if request.auth != null;
    }
    // Todo o resto (financeiro): só a equipe logada
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```
