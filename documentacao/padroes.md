# 1. Análise dos Princípios SOLID
## S — Single Responsibility Principle (SRP)

Cada módulo deve ter uma única responsabilidade.
No sistema, há separação básica entre:

- **Controllers** → tratam requisições/respostas.

- **Models** → definem os esquemas do MongoDB (via Mongoose).

- **Services** → concentram lógica de negócio.

Isso já aproxima o projeto do SRP, mas em alguns pontos controllers acumulam validações e regras de negócio.

Exemplo:
```js
  // controller (fino)
  class CreateUserController {
      constructor(private readonly createUser: CreateUserService) {}

      async handle(request: Request, response: Response): Promise<Response> {
        const { name, email, password } = request.body;
        try {
          const result = await this.createUser.execute({ name, email, password }); // Delega para o serviço
          const statusCode = result.isFailure() ? 400 : 201;
          return response.status(statusCode).json(result.value);
        } catch (err) {
          const error = err as Error;
          console.log({ error });
          return response.status(500).json({ error: error.message });
        }
      }
    }
```
## O — Open/Closed Principle (OCP)

O código deve estar aberto para extensão, mas fechado para modificação.
Exemplo no projeto: é possível adicionar novos endpoints sem alterar os existentes.
Sugestão: aplicar estratégias de filtro ou validação para estender funcionalidades sem mudar código já pronto.

## L — Liskov Substitution Principle (LSP)

Subtipos devem poder substituir seus tipos base sem alterar o comportamento.
Embora o código seja em JavaScript (sem herança formal), podemos aplicar esse princípio quando definimos contratos claros (ex.: serviços que sempre retornam `Promise`).

## I — Interface Segregation Principle (ISP)

Prefira interfaces pequenas a interfaces grandes.
Na prática, em vez de um “mega service”, é melhor dividir em serviços específicos (ex.: `PetService`, `UserService`).

## D — Dependency Inversion Principle (DIP)

Módulos de alto nível não devem depender diretamente de implementações de baixo nível.
Hoje, alguns services usam direto o `PetModel` do Mongoose.
Uma melhoria seria criar uma abstração de repositório que permita trocar a persistência sem mudar o domínio.

# 2. Padrões de Projeto
## 2.1 Singleton — Instância Única de um Repositório

```js
// db.js
export const userRepositoryInstance = new UserRepository(prisma)
```

## 2.2 Strategy — Regras de Negócio Variáveis

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
