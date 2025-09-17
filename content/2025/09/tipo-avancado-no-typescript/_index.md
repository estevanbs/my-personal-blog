---
title: Tipo avançado no TypeScript
toc: false
---

Recentemente criei um tipo relativamente complexo no TypeScript e vou explicar ele aqui. Não consegui encontrar qual é o nome oficial dele. Se você souber, não deixe de comentar!

Como tudo que exige um pouco mais de raciocínio quando faço, vou quebrar a explicação em partes. Os exemplos utilizados aqui não são os mesmos que usei quando desenvolvi a lógica pela primeira vez, mas com eles fica mais simples para entender.

Primeiro criei o tipo `Person`, com as propriedades `name`, `age` e `address`.

```
type Person = {
    name: string
    age: number
    address: {
        street: string
        number: number
    }
}
```

Depois defini o que eu preciso. Variáveis do meu tipo devem guardar o valor de alguma propriedade do tipo `Person`. A propriedade que ele guarda deve ser definida em `property`, e o valor em `value`. Chamei esse tipo de `PersonPropertyValue`.

Por exemplo: 

```
// valid
const object: PersonPropertyValue = {
    propertyValue: {
        property: 'age',
        value: 2
    }
}
```

ou

```
// valid
const object: PersonPropertyValue = {
    propertyValue: {
        property: 'address',
        value: {
            street: 'Saint Street',
            number: 3
        }
    }
}
```

devem ser válidos. A constante:

```
// invalid
const object: PersonPropertyValue = {
    propertyValue: {
        property: 'address',
        value: 'Saint Street'
    }
}
```

deve ser inválida, pois address tem o valor de um objeto, e não de uma string, e o typescript deve considerar um erro em tempo de compilação.

Como fazer isso de maneira que property e value sejam dinâmicas e utilizem o tipo Person como referência?

```
type PersonPropertyValue = {
    propertyValue: {
        [K in keyof Person]: {
            property: K
            value: Person[K]
        }
    }[keyof Person]
}
```

Funciona! Mas como funciona? O trecho `[K in keyof Person]` faz um loop em todas as propriedades de person, e no nosso caso, para cada propriedade, ele cria um objeto seguindo o que foi declarado na sequência. Aqui aconteceu o seguinte:

```
propertyValue: {
    name: {
        property: "name";
        value: string;
    };
    age: {
        property: "age";
        value: number;
    };
    address: {
        property: "address";
        value: {
            street: string;
            number: number;
        };
    };
}
```

Mas não é isso que precisamos. Queremos que propertyValue **"entre"** em cada uma de suas propriedades. Então utilizamos o `[keyof Person]`. Com ele, cada propriedade de propertyValue é acessada e acaba sendo removida do nosso tipo, mas seus valores continuam aqui. Acabamos com o seguinte resultado:

```
type PersonPropertyValue = {
    propertyValue: {
        property: "name";
        value: string;
    } |
    {
        property: "age";
        value: number;
    } |
    {
        property: "address";
        value: {
            street: string;
            number: number;
        };
    };
}
```

Que é exatamente o que precisamos. E o melhor, isso está utilizando o tipo `Person` como referência. Qualquer alteração nele será refletido aqui.
