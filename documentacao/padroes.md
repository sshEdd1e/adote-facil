#1. Análise dos Princípios SOLID
##S — Single Responsibility Principle (SRP)

Cada módulo deve ter uma única responsabilidade.
No sistema, há separação básica entre:

- **Controllers** → tratam requisições/respostas.

- **Models** → definem os esquemas do MongoDB (via Mongoose).

- **Services** → concentram lógica de negócio.

Isso já aproxima o projeto do SRP, mas em alguns pontos controllers acumulam validações e regras de negócio.

Exemplo:
```js
  // controller (fino)
  async function createPet(req, res, next) {
    try {
      const pet = await petService.create(req.body);
      res.status(201).json(pet);
    } catch (e) { next(e); }
  }
```
##O — Open/Closed Principle (OCP)

O código deve estar aberto para extensão, mas fechado para modificação.
Exemplo no projeto: é possível adicionar novos endpoints sem alterar os existentes.
Sugestão: aplicar estratégias de filtro ou validação para estender funcionalidades sem mudar código já pronto.

##L — Liskov Substitution Principle (LSP)

Subtipos devem poder substituir seus tipos base sem alterar o comportamento.
Embora o código seja em JavaScript (sem herança formal), podemos aplicar esse princípio quando definimos contratos claros (ex.: serviços que sempre retornam `Promise`).

##I — Interface Segregation Principle (ISP)

Prefira interfaces pequenas a interfaces grandes.
Na prática, em vez de um “mega service”, é melhor dividir em serviços específicos (ex.: `PetService`, `UserService`).

##D — Dependency Inversion Principle (DIP)

Módulos de alto nível não devem depender diretamente de implementações de baixo nível.
Hoje, alguns services usam direto o `PetModel` do Mongoose.
Uma melhoria seria criar uma abstração de repositório que permita trocar a persistência sem mudar o domínio.

#2. Padrões de Projeto
##2.1 Singleton — Conexão com o Banco

O sistema deve manter apenas uma instância de conexão com o MongoDB.
Esse é um exemplo clássico de Singleton.

```js
// db.js
const mongoose = require('mongoose');
let instance = null;

async function connect(uri) {
  if (!instance) {
    instance = await mongoose.connect(uri, { dbName: 'adote' });
  }
  return instance;
}

module.exports = { connect };
```

##2.2 Strategy — Regras de Negócio Variáveis

Permite definir diferentes estratégias sem mudar o código cliente.
Pode ser aplicado, por exemplo, em filtros de listagem de pets.

```js
class AgeFilter {
  apply(query, { minAge }) {
    return minAge ? query.where('age').gte(minAge) : query;
  }
}

class SpeciesFilter {
  apply(query, { species }) {
    return species ? query.where('species', species) : query;
  }
}

// uso
const filters = [new AgeFilter(), new SpeciesFilter()];
let query = PetModel.find();
filters.forEach(f => { query = f.apply(query, req.query); });
```
