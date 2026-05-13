# Respostas do Exercício Prático: Modelagem e Consultas em MongoDB

Este arquivo contém as respostas para o exercício proposto, consolidando os conceitos de modelagem NoSQL (Incorporação vs. Referência) e o uso do Framework de Agregação.

---

## Parte 1: Modelagem de Dados
Imagine que você está desenvolvendo um sistema para uma **Plataforma de Cursos Online**.

### 1. Incorporação (Embedding)
Um exemplo de documento para um curso com módulos incorporados seria:

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "titulo": "Curso de MongoDB Básico",
  "descricao": "Introdução aos conceitos fundamentais do MongoDB",
  "modulos": [
    {
      "nome": "Introdução ao NoSQL",
      "carga_horaria": 4
    },
    {
      "nome": "Modelagem de Dados",
      "carga_horaria": 6
    },
    {
      "nome": "Consultas e Agregações",
      "carga_horaria": 8
    }
  ]
}
```

**Pergunta:** Por que, neste caso, colocar os módulos dentro do documento do curso (incorporar) é melhor para o desempenho de leitura?  
**Resposta:** Incorporar os módulos dentro do documento do curso é melhor para o desempenho de leitura porque permite recuperar todos os dados relacionados (curso e módulos) em uma única consulta ao banco de dados, sem a necessidade de operações de "join" ou consultas adicionais. Isso reduz a latência e melhora a eficiência, especialmente quando os módulos são sempre acessados junto com o curso e não mudam frequentemente de forma independente.

### 2. Referência (Reference)
Um exemplo de documentos separados seria:

Documento para o Instrutor:
```json
{
  "_id": ObjectId("507f1f77bcf86cd799439012"),
  "nome": "Maria Silva",
  "especialidade": "Banco de Dados",
  "email": "maria.silva@exemplo.com"
}
```

Documento para o Curso:
```json
{
  "_id": ObjectId("507f1f77bcf86cd799439013"),
  "titulo": "Curso Avançado de MongoDB",
  "descricao": "Técnicas avançadas de modelagem e otimização",
  "instrutor_id": ObjectId("507f1f77bcf86cd799439012")
}
```

**Pergunta:** Em que situação seria preferível referenciar o instrutor em vez de copiar os dados dele dentro de cada curso que ele ministra?  
**Resposta:** Seria preferível referenciar o instrutor quando ele é compartilhado por vários cursos, para evitar duplicação de dados e manter a consistência (por exemplo, se o email ou especialidade do instrutor mudar, basta atualizar um documento). Também é útil quando os dados do instrutor são acessados independentemente dos cursos ou quando há necessidade de relacionamentos mais complexos, como um instrutor ministrando muitos cursos.

---

## Parte 2: Consultas e Filtros
Utilizando a coleção `clientes` com operadores MongoDB:

1. **Filtro de Existência:**  
   Comando: `db.clientes.find({ email: { $exists: true } })`  
   Este comando encontra todos os documentos na coleção `clientes` que possuem o campo `email` preenchido (ou seja, o campo existe e não é nulo).

2. **Filtro de Comparação:**  
   Comando: `db.clientes.find({ idade: { $gte: 21 } })`  
   Este comando encontra todos os clientes com `idade` maior ou igual a 21 anos.

3. **Filtro Complexo:**  
   Comando: `db.clientes.find({ cidade: "São Paulo", idade: { $lt: 30 } })`  
   Este comando busca clientes que moram na cidade "São Paulo" **E** têm idade inferior a 30 anos, combinando os operadores de igualdade e comparação.

---

## Parte 3: Aggregation Framework (Relatórios)
Para a coleção `vendas` com a estrutura de exemplo `{ produto: "Webcam", categoria: "Periféricos", quantidade: 3, preco_unitario: 200.00, status: "concluída" }`, o pipeline de agregação para gerar o relatório solicitado seria:

```javascript
db.vendas.aggregate([
  {
    $match: { status: "concluída" }
  },
  {
    $group: {
      _id: "$categoria",
      total_quantidade: { $sum: "$quantidade" }
    }
  }
])
```

**Explicação do pipeline:**
1. **$match:** Filtra apenas os documentos onde `status` é igual a "concluída".
2. **$group:** Agrupa os documentos pelo campo `categoria` (usando `_id: "$categoria"`) e calcula a soma total da `quantidade` para cada grupo usando `$sum: "$quantidade"`.

O resultado será algo como:
```json
[
  { "_id": "Periféricos", "total_quantidade": 15 },
  { "_id": "Eletrônicos", "total_quantidade": 8 }
]
```

Isso gera um relatório com a soma da quantidade vendida por categoria, apenas para vendas concluídas.