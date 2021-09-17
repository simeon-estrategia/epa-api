## 1. Filtros diretos

Cada campo da tabela já consegue ser filtrado sem nenhum intervenção no código. Se a coluna existe no banco, ela já possui um filtro correspondente.
EX: tabela: users

| id   | nome          | sexo |
| ---- | ------------- | ---- |
| 15   | Maria Bolacha | F    |
| 16   | João Biscoito | M    |

`/users?codigo=15` irá trazer apenas os usuário com o código 15, neste caso a `Maria Bolacha`.

#### Opções de filtros diretos

| Nome               | Caractere                                    | Descrição                                                 | Exemplo                    |
| ------------------ | -------------------------------------------- | --------------------------------------------------------- | -------------------------- |
| Igualdade          | =                                            | Busca os registros com o valor igual ao informado         | /users?codigo=15           |
| Diferença          | !=                                           | Busca os registros com o valor direferente do informado   | /users?codigo!=15          |
| Contido            | :LIKE= OU :\*LIKE= OU :LIKE\*= OU :\*LIKE\*= | Busca os registros que o valor informado esteja contido   | /users?nome:\*LIKE\*=Maria |
| Maior que          | >=                                           | Busca os registros com o valor maior que o informado      | /users?codigo>=15          |
| Menor que          | <=                                           | Busca os registros com o valor menor que o informado      | /users?codigo<=15          |
| Maior ou igual que | >-=                                          | Busca os registros com o valor maior ou igual o informado | /users?codigo>-=15         |
| Menor ou igual que | <-=                                          | Busca os registros com o valor menor ou igual o informado | /users?codigo<-=15         |

Obs: Para todos os operadores exceto o `Contido` é possível passar vários valores separados por `,`;

EX: `users?codigo=15,2,62,105`

Obs2: Os filtros de data funcionam de forma diferente. Verifique o item 3;






## 2. Filtros de Relacionamento  [Ainda não implementado]

Filtros de relacionamento se baseiam em filtrar a lista de items baseado em um regra do seu relacionamento ou filtros os relacionamentos do item.



Vamos imaginar que temos as duas tabelas abaixo onde um usuário pode ter vários telefones:

Users:

| id   | nome          |
| ---- | ------------- |
| 1    | Maria Bolacha |
| 2    | João Biscoito |



Phones: 

| id   | ddd  | number     | user_id |
| ---- | ---- | ---------- | ------- |
| 1    | 62   | 91234-5678 | 1       |
| 2    | 64   | 99874-6541 | 1       |
| 3    | 62   | 91155-9201 | 2       |



É possível utilizar o filtro de realcionamento de duas formas

1. Filtrar a quantidade de usuários retornados aplicando a regra de filtro nos telefones com o `parent`;

   EX:

   **Requisição**: `/users?with=telefones&telefones:ddd=64` ou `/users?with=telefones&telefones:parent:ddd=64`

   **Resultado**: 

   ```json
   [
       {
       	"id": 1,
           "nome": "Maria Bolacha",
           "telefones": [
               {
                   "id": 1,
                   "ddd": 62,
                   "number": "91234-5678",
                   "user_id": 1,
               },
               {
                   "id": 2,
                   "ddd": 64,
                   "number": "99874-6541",
                   "user_id": 1,
               }
           ]
   	}
   ]
   ```



2. Filtrar a quantidade de telefones a aplicando a regra de filtro nos telefones com o `child`. Neste cenário o parametro `with` é opcional:

   EX:

   **Requisição**: `/users?telefones:child:ddd=64`

   **Resultado**: 

   ```json
   [
       {
       	"id": 1,
           "nome": "Maria Bolacha",
           "telefones": [
               {
                   "id": 2,
                   "ddd": 64,
                   "number": "99874-6541",
                   "user_id": 1,
               }
           ]
   	},
       {
       	"id": 2,
           "nome": "João Biscoito",
           "telefones": []
   	},
   ]
   ```

   

   

   

## 3. Filtros de Data

- Um campo é considerado um capo de data, quando começa com `data`;

​       EX: `data_inclusao`, `datafim`;



- Qualquer campo de data pode ser filtrado utilizando o formato `dia/mês/ano` ou `ano-mes-dia`;

- Os campos de data podem ser filtrados de 4 formas:

  - Filtro a partir de uma data

    EX: `/endpoint?data_inclusao=12/12/2012`

  - Até uma data utilizando o separador `to` 

    EX: `/endpoint?data_inclusao=to12/12/2012`

  - Dentro de um período utilizando o separador `to` 

    EX: `/endpoint?data_inclusao=12/12/2012to13/13/2013`



OBS: Caso queira realizar um filtro de período entre dois campos de datas diferentes, é necessário colocar o separador `to` na data que representa a data final

​	EX: `/endpoint?data_inicio=12/12/2012&data_fim=to13/13/2013`







## 4. onlyColumns [Ainda não implementado]

`onlyColumn` é um parâmetro que tem o objetivo diminuir a quatidade de dados que vêm na resposta otimizando a busca. Com este parâmetro é possível selecionar quais campos devem ser retornados na resposta.

EX (Sem `onlyColumns`):

**Requisição**:  /users

**Resultado**: 

```json
[
    {
    	"id": 15,
        "nome": "Maria Bolacha",
        "sexo": "F"
	},
    {
    	"id": 16,
        "nome": "João Biscoito",
        "sexo": "M"
	},
]
```



EX (Com `onlyColumns`):

**Requisição**:  /users?onlyColumns=nome,id

**Resultado**: 

```json
[
    {
    	"id": 15,
        "nome": "Maria Bolacha"
	},
    {
    	"id": 16,
        "nome": "João Biscoito"
	},
]
```



Também é possível filtrar os relacionamentos de um item caso aja relacionamento

EX: `/users?onlyColumns=nome,id,endereco:rua,cidade`



## 5. hasText

`hasText` tem o objetivo de filtrar com `LIKE` os items de um endpoint.

Para que o `hasText` funcione é necessário sobrescrever o attributo `protected $hasTextFields = [];` na lista. Dentro do array do `$hasTextFields` devem ser informados todos os campos de text da tabela.

Ao realizar um requisição com o `hasText` o filtro será de `LIKE` do valor informado em qualquer campo informado no `hasText`

Vamos imaginar que temos a tabelas abaixo com o seguinte has text: `protected $hasTextFields = ['rua', 'cidade'];`

| id   | rua               | cidade        |
| ---- | ----------------- | ------------- |
| 1    | Rua  Goiânia      | Goiânia       |
| 2    | Rua  São Paulo    | Goiânia       |
| 3    | Rua Florianópolis | Florianópolis |

**Requisição**:  /addresses?hasText=Goiânia

**Resultado**: 

```json
[
    {
    	"id": 1,
        "nome": "Rua Goiânia",
        "cidade": "Goiania"
	},
    {
    	"id": 2,
        "nome": "Rua São Paulo",
        "cidade": "Goiania"
	},
]
```







## 6. with

`hasText` é utilizado com o objetivo de trazer dados a mais na resposta. Dados estes que estão relacionados ao items da resposta.



Vamos imaginar que temos as duas tabelas abaixo onde um usuário pode ter vários telefones:

Users:

| id   | nome          |
| ---- | ------------- |
| 1    | Maria Bolacha |
| 2    | João Biscoito |



Telefones: 

| id   | ddd  | number     | user_id |
| ---- | ---- | ---------- | ------- |
| 1    | 62   | 91234-5678 | 1       |
| 2    | 64   | 99874-6541 | 1       |
| 3    | 62   | 91155-9201 | 2       |





Para que o with funcione, é preciso que o relacionamento esteja corretamente especificado dentro do model.

```php
<?php
    
use App\EpaBaseModel;
use Illuminate\Database\Eloquent\Relations\HasMany;
    
class UserModel extends EpaBaseModel
{
    public function telefones(): HasMany
    {
        return $this->hasMany(Telefone::class, 'user_id', 'id');
    }
}
```



Com isso, podemos adicionar o parâmetro with na requisição para trazer os dados dos telefones do usuário

**Requisição**:  /users?with=telefones

**Resultado**: 

```json
[
    {
    	"id": 1,
        "nome": "Maria Bolacha",
        "telefones": [
            {
                "id": 1,
                "ddd": 62,
                "number": "91234-5678",
                "user_id": 1,
            },
            {
                "id": 2,
                "ddd": 64,
                "number": "99874-6541",
                "user_id": 1,
            },
        ]
	},
    {
    	"id": 2,
        "nome": "João Biscoito",
        "telefones": [
            {
                "id": 3,
                "ddd": 62,
                "number": "91155-9201",
                "user_id": 2,
            },
        ]
	},
]
```

