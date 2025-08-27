# Code Smells e Refatorações
## SonarQube
Utilizei a ferramenta `SonarQube` Cloud para realizar a análise de `code smells`. O SonarQube identificou **36 issues** do tipo code smell, sendo 2 de gravidade de informação, 20 de gravidade baixa, 9 de gravidade média e 5 de gravidade tanto baixo quanto média.
![Resultados SonarQube](img/SonarQube.png)

## Caso 1 - Campo não marcado como readonly
**Código original**
```js 
//backend/src/providers/authenticator.ts
import jwt from 'jsonwebtoken'

export class Authenticator {
  private secret = process.env.JWT_SECRET || 'secret'

  generateToken(payload: object): string {
    return jwt.sign(payload, this.secret, { expiresIn: '1h' })
  }

  validateToken<T = object>(token: string): T | null {
    const secret = process.env.JWT_SECRET || 'secret'

    try {
      return jwt.verify(token, secret) as T
    } catch (err) {
      const error = err as Error
      console.log({ error })
      return null
    }
  }
}

export const authenticatorInstance = new Authenticator()
```

**Smell identificado**

Imutability Violation / Field not marked as readonly
O campo secret é atribuído apenas uma vez e nunca é reatribuído.
Manter o campo como mutável gera confusão sobre sua intenção e pode induzir futuros desenvolvedores a pensar que ele pode ser modificado.

**Refatoração aplicada**

Marcar o campo como readonly torna explícito que seu valor não muda durante a vida da instância.

```js
import jwt from 'jsonwebtoken'

export class Authenticator {
  private readonly secret = process.env.JWT_SECRET || 'secret'

  generateToken(payload: object): string {
    return jwt.sign(payload, this.secret, { expiresIn: '1h' })
  }

  validateToken<T = object>(token: string): T | null {
    const secret = process.env.JWT_SECRET || 'secret'

    try {
      return jwt.verify(token, secret) as T
    } catch (err) {
      const error = err as Error
      console.log({ error })
      return null
    }
  }
}

export const authenticatorInstance = new Authenticator()
```
**Benefícios da refatoração**

- `Clareza: indica explicitamente que o campo é imutável.`

- `Segurança: evita modificações acidentais no valor de secret.`

## Caso 2 - Operador ternário aninhado
**Código original**
```js 
//frontend/src/app/area_logada/animais_disponiveis/AvailableAnimalsPage.tsx
) : availableAnimals.length ? (
        <S.AnimalsListWrapper>
          {availableAnimals.map((animal) => (
            <AnimalCard
              key={animal.id}
              animal={animal}
              listType="animals-available-to-adopt"
            />
          ))}
        </S.AnimalsListWrapper>
      ) : (
        <EmptyAnimals page="animals-available-to-adopt" />
      )}
```

**Smell identificado**

Nested Ternary / Complex Conditional Rendering 
Ternários aninhados no JSX dificultam leitura e manutenção.

**Refatoração aplicada**

Extração do fluxo condicional para declaração independente antes do return, eliminando o ternário aninhado e mantendo o mesmo comportamento.
```js
function renderContent(loading: boolean, animals: Animal[]) {
  if (loading) return <p>Carregando...</p>
  if (animals.length > 0) {
    return (
      <S.AnimalsListWrapper>
        {animals.map((animal) => (
          <AnimalCard
            key={animal.id}
            animal={animal}
            listType="animals-available-to-adopt"
          />
        ))}
      </S.AnimalsListWrapper>
    )
  }
  return <EmptyAnimals page="animals-available-to-adopt" />
}
```

## Caso 3 - Array index em chaves
**Código original**
```js
//frontend/src/app/area_logada/animais_disponiveis/[id]/AnimalDetailsPage.tsx
{animal.images.map((image, index) => (
    <S.AnimalPictureSwiperSlide key={index}>
    <Image
        src={`data:image/jpeg;base64,${image}`}
        alt="Animal"
        fill={true}
        objectFit="cover"
    />
    </S.AnimalPictureSwiperSlide>
))}
```
**Smell identificado**

Array index as key 
Uso do índice do array como key em lista React pode gerar remounts incorretos quando a ordem muda, afetando desempenho e estado.

**Refatoração aplicada**

Substituição de key={index} por uma key estável derivada do próprio item (key={image}). Mantém a identidade do item entre renders e atende a recomendação do SonarQube.
```js
{animal.images.map((image) => (
  <S.AnimalPictureSwiperSlide key={image}>
    <Image
      src={`data:image/jpeg;base64,${image}`}
      alt="Animal"
      fill={true}
      objectFit="cover"
    />
  </S.AnimalPictureSwiperSlide>
))}
```