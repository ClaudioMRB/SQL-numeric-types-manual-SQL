# SQL-numeric-types-manual-
Esta documentação é para uma versão sem suporte do PostgreSQL.

[PostgreSQL: Documentação: 9.1: Tipos numéricos](https://www.postgresql.org/docs/9.1/datatype-numeric.html)



# 8.1. Tipos numéricos

Os tipos numéricos consistem em inteiros de dois, quatro e oito byte, números de pontos flutuantes de quatro e oito byte, e decimais de precisão selecionável. [A Tabela 8-2](https://www.postgresql.org/docs/9.1/datatype-numeric.html#DATATYPE-NUMERIC-TABLE) lista os tipos disponíveis.



Tabela 8-2. Tipos numéricos

| Nome             | Tamanho do armazenamento | Descrição                                 | Gama                                                         |
| ---------------- | ------------------------ | ----------------------------------------- | ------------------------------------------------------------ |
| `smallint`       | 2 bytes                  | inteiro de pequena gama                   | -32768 a +32767                                              |
| `inteiro`        | 4 bytes                  | escolha típica para inteiro               | -2147483648 para +2147483647                                 |
| `bigint`         | 8 bytes                  | inteiro de grande alcance                 | - 9223372036854775808 para 9223372036854775807               |
| `decimal`        | variável                 | precisão especificada pelo usuário, exata | até 131072 dígitos antes do ponto decimal; até 16383 dígitos após o ponto decimal |
| `numérico`       | variável                 | precisão especificada pelo usuário, exata | até 131072 dígitos antes do ponto decimal; até 16383 dígitos após o ponto decimal |
| `real`           | 4 bytes                  | variável-precisão, inexato                | Precisão de 6 dígitos decimais                               |
| `dupla precisão` | 8 bytes                  | variável-precisão, inexato                | Precisão de 15 dígitos decimais                              |
| `serial`         | 4 bytes                  | inteiro autoincrementante                 | 1 a 2147483647                                               |
| `grandeseria`    | 8 bytes                  | inteiro autoincrementando inteiro         | 1 a 9223372036854775807                                      |

A sintaxe das constantes para os tipos numéricos está descrita na [Seção 4.1.2](https://www.postgresql.org/docs/9.1/sql-syntax-lexical.html#SQL-SYNTAX-CONSTANTS). Os tipos numéricos possuem um conjunto completo de operadores e funções aritméticas correspondentes. Consulte o [Capítulo 9](https://www.postgresql.org/docs/9.1/functions.html) para obter mais informações. As seções a seguir descrevem os tipos em detalhes.

## 8.1.1. Tipos inteiros

Os tipos `smallint`, `inteiros` e `bigint` armazenam números inteiros, ou seja, números sem componentes fracionados, de várias faixas. Tentativas de armazenar valores fora do intervalo permitido resultarão em um erro.

O `inteiro tipo` é a escolha comum, pois oferece o melhor equilíbrio entre alcance, tamanho de armazenamento e desempenho. O tipo `smallint` geralmente só é usado se o espaço em disco estiver em um prêmio. O tipo `bigint` só deve ser usado se a gama do tipo `inteiro` for insuficiente, porque este último é definitivamente mais rápido.

Em sistemas operacionais muito mínimos, o tipo `bigint` pode não funcionar corretamente, porque depende do suporte do compilador para inteiros de oito byte. Em tais máquinas, `bigint` age o mesmo `que inteiro`, mas ainda ocupa oito bytes de armazenamento. (Não estamos cientes de nenhuma plataforma moderna onde este é o caso.)

SQL só especifica os tipos inteiros `inteiros inteiros` (ou `int`), `smallint` e `bigint`. Os nomes de digitar `int2`, `int4` e `int8` são extensões, que também são usadas por alguns outros sistemas de banco de dados SQL.

## 8.1.2. Números de Precisão Arbitrária

O tipo `numérico` pode armazenar números com um número muito grande de dígitos e realizar cálculos exatamente. É especialmente recomendado para armazenar quantidades monetárias e outras quantidades onde a exatidão é necessária. No entanto, a aritmética sobre valores `numéricos` é muito lenta em comparação com os tipos inteiros, ou com os tipos de pontos flutuantes descritos na próxima seção.

Usamos os seguintes termos abaixo: A *escala* de um `numérico` é a contagem de dígitos decimais na parte fracionada, à direita do ponto decimal. A *precisão* de um `numérico` é a contagem total de dígitos significativos em todo o número, ou seja, o número de dígitos para ambos os lados do ponto decimal. Assim, o número 23.5141 tem uma precisão de 6 e uma escala de 4. Inteiros podem ser considerados com uma escala de zero.

Tanto a máxima precisão quanto a escala máxima de uma coluna `numérica` podem ser configuradas. Para declarar uma coluna de tipo `numérico` use a sintaxe:

```
NUMERIC(precision, scale)
```

A precisão deve ser positiva, a escala zero ou positiva. Alternativamente:

```
NUMERIC(precision)
```

seleciona uma escala de 0. Especificando:

```
NUMERIC
```

sem qualquer precisão ou escala cria uma coluna na qual valores numéricos de qualquer precisão e escala podem ser armazenados, até o limite de implementação em precisão. Uma coluna deste tipo não coagirá valores de entrada a nenhuma escala específica, enquanto colunas `numéricas` com escala declarada coagirão valores de entrada a essa escala. (O padrão SQL requer uma escala padrão de 0, ou seja, coerção à precisão de inteiro. Achamos isso um pouco inútil. Se você está preocupado com a portabilidade, sempre especifique a precisão e a escala explicitamente.)

> **Nota:** A precisão máxima permitida quando explicitamente especificada na declaração de tipo é 1000; `Numeric` sem uma precisão especificada está sujeito aos limites descritos na [Tabela 8-2](https://www.postgresql.org/docs/9.1/datatype-numeric.html#DATATYPE-NUMERIC-TABLE).

Se a escala de um valor a ser armazenado for maior do que a escala declarada da coluna, o sistema arredondará o valor para o número especificado de dígitos fracionados. Então, se o número de dígitos à esquerda do ponto decimal exceder a precisão declarada menos a escala declarada, um erro é levantado.

Os valores numéricos são armazenados fisicamente sem nenhum número extra de pontos mento ou zeros. Assim, a precisão declarada e a escala de uma coluna são máximos, não alocações fixas. (Nesse sentido, o tipo `numérico` é mais parecido com `varchar(n)` do que `com char(n)`.) O requisito real de armazenamento é de dois bytes para cada grupo de quatro dígitos decimais, mais três a oito bytes overhead.

Além dos valores numéricos comuns, o tipo `numérico` permite o valor especial `NaN`, que significa "não-um-número". Qualquer operação na `NaN` rende outra `NaN`. Ao escrever esse valor como uma constante em um comando SQL, você deve colocar as cotações em torno dele, por exemplo`, atualizar tabela SET x = 'NaN'`. Na entrada, a `string NaN` é reconhecida de forma insensível a caso.

> **Nota:** Na maioria das implementações do conceito "não-um-número", `a NaN` não é considerada igual a qualquer outro valor numérico (incluindo `o NaN`). Para permitir que valores `numéricos` sejam classificados e usados em índices baseados em árvores, o PostgreSQL trata os valores `da NaN` como iguais e maiores do que todos os valores `não-NaN`.

Os tipos `decimais` e `numéricos` são equivalentes. Ambos os tipos fazem parte do padrão SQL.

## 8.1.3. Tipos de pontos flutuantes

Os tipos de dados `reais` e `de dupla precisão` são tipos numéricos de precisão variável e inexatos. Na prática, esses tipos geralmente são implementações do IEEE Standard 754 para Aritmética Binária flutuante de ponto flutuante (precisão única e dupla, respectivamente), na medida em que o processador, sistema operacional e compilador subjacentes o suportam.

Inexata significa que alguns valores não podem ser convertidos exatamente para o formato interno e são armazenados como aproximações, de modo que armazenar e recuperar um valor pode apresentar pequenas discrepâncias. Gerenciar esses erros e como eles se propagam através de cálculos é objeto de todo um ramo de matemática e ciência da computação e não será discutido aqui, exceto pelos seguintes pontos:

- Se você precisar de armazenamento e cálculos exatos (como para valores monetários), use o tipo `numérico` em vez disso.
- Se você quiser fazer cálculos complicados com esses tipos para qualquer coisa importante, especialmente se você confiar em certos comportamentos em casos de limite (infinito, subfluxo), você deve avaliar a implementação com cuidado.
- Comparar dois valores de pontos flutuantes para a igualdade pode nem sempre funcionar como esperado.

Na maioria das plataformas, o tipo `real` tem uma gama de pelo menos 1E-37 a 1E+37 com uma precisão de pelo menos 6 dígitos decimais. O tipo de `dupla precisão` normalmente tem um alcance de cerca de 1E-307 a 1E+308 com uma precisão de pelo menos 15 dígitos. Valores muito grandes ou muito pequenos causarão um erro. O arredondamento pode ocorrer se a precisão de um número de entrada for muito alta. Números muito próximos a zero que não são representaveis como distintos de zero causarão um erro de subfluxo.

> **Nota:** A configuração [extra_float_digits](https://www.postgresql.org/docs/9.1/runtime-config-client.html#GUC-EXTRA-FLOAT-DIGITS) controla o número de dígitos significativos extras incluídos quando um valor de ponto flutuante é convertido em texto para saída. Com o valor padrão de `0`, a saída é a mesma em todas as plataformas suportadas pelo PostgreSQL. Aumentando-o produzirá uma saída que represente com mais precisão o valor armazenado, mas pode ser inportável.

Além dos valores numéricos comuns, os tipos de pontos flutuantes têm vários valores especiais:

```
Infinity`
`-Infinity`
`NaN
```

Estes representam os valores especiais IEEE 754 "infinito", "infinito negativo" e "não-um-número", respectivamente. (Em uma máquina cuja aritmética de ponto flutuante não segue o IEEE 754, esses valores provavelmente não funcionarão como esperado.) Ao escrever esses valores como constantes em um comando SQL, você deve colocar aspas ao redor delas, por exemplo, `tabela UPDATE SET x = 'Infinity'`. Na entrada, essas cordas são reconhecidas de forma insensível a casos.

> **Nota:** O IEEE754 especifica que `a NaN` não deve comparar igual a qualquer outro valor de ponto flutuante (incluindo `NaN`). Para permitir que valores de pontos flutuantes sejam classificados e usados em índices baseados em árvores, o PostgreSQL trata os valores `da NaN` como iguais e maiores do que todos os valores `não-NaN`.

PostgreSQL also supports the SQL-standard notations `float` and `float(p)` for specifying inexact numeric types. Here, `p` specifies the minimum acceptable precision in binary digits. PostgreSQL accepts `float(1)` to `float(24)` as selecting the `real` type, while `float(25)` to `float(53)` select `double precision`. Values of `p` outside the allowed range draw an error. `float` with no precision specified is taken to mean `double precision`.

> **Note:** Prior to PostgreSQL 7.4, the precision in `float(p)` was taken to mean so many decimal digits. This has been corrected to match the SQL standard, which specifies that the precision is measured in binary digits. The assumption that `real` and `double precision` have exactly 24 and 53 bits in the mantissa respectively is correct for IEEE-standard floating point implementations. On non-IEEE platforms it might be off a little, but for simplicity the same ranges of `p` are used on all platforms.

## 8.1.4. Serial Types

The data types `serial` and `bigserial` are not true types, but merely a notational convenience for creating unique identifier columns (similar to the `AUTO_INCREMENT` property supported by some other databases). In the current implementation, specifying:

```
CREATE TABLE tablename (
    colname SERIAL
);
```

is equivalent to specifying:

```
CREATE SEQUENCE tablename_colname_seq;
CREATE TABLE tablename (
    colname integer NOT NULL DEFAULT nextval('tablename_colname_seq')
);
ALTER SEQUENCE tablename_colname_seq OWNED BY tablename.colname;
```

Thus, we have created an integer column and arranged for its default values to be assigned from a sequence generator. A `NOT NULL` constraint is applied to ensure that a null value cannot be inserted. (In most cases you would also want to attach a `UNIQUE` or `PRIMARY KEY` constraint to prevent duplicate values from being inserted by accident, but this is not automatic.) Lastly, the sequence is marked as "owned by" the column, so that it will be dropped if the column or table is dropped.

> **Note:** Because `smallserial`, `serial` and `bigserial` are implemented using sequences, there may be "holes" or gaps in the sequence of values which appears in the column, even if no rows are ever deleted. A value allocated from the sequence is still "used up" even if a row containing that value is never successfully inserted into the table column. This may happen, for example, if the inserting transaction rolls back. See `nextval()` in [Section 9.15](https://www.postgresql.org/docs/9.1/functions-sequence.html) for details.

> **Nota:** Antes do PostgreSQL 7.3, `a série` implicava `UNIQUE`. Isso não é mais automático. Se você deseja que uma coluna serial tenha uma restrição única ou seja uma chave primária, ela agora deve ser especificada, assim como qualquer outro tipo de dados.

Para inserir o próximo valor da sequência na coluna `serial`, especifique que a coluna `serial` deve ser atribuída ao seu valor padrão. Isso pode ser feito excluindo a coluna da lista de colunas na instrução `INSERT` ou através do uso da palavra-chave `PADRÃO`.

Os nomes `de tipo serial` e `serial4` são equivalentes: ambos criam colunas `inteiras`. Os nomes do tipo `bigserial` e `serial8` funcionam da mesma maneira, exceto que eles criam uma coluna `bigint`. `bigserial` deve ser usado se você antecipar o uso de mais de 231 identificadores ao longo da vida da tabela.

A sequência criada para uma coluna `serial` é automaticamente descartada quando a coluna proprietária é descartada. Você pode soltar a sequência sem soltar a coluna, mas isso forçará a remoção da expressão padrão da coluna.
