# Danny's Diner
 _Challenge #1 from 8 Week SQL Challenge_ 

# 1. Introdução

 Esse projeto é o primeiro desafio do programa 8 Week SQL Challenge do Data With Danny.

 Nesse projeto, o trabalho do analista de dados é solicitado para auxiliar o dono de um restaurante a compreender dados básicos do seu negócio.

 O proprietário quer usar os dados para responder a algumas perguntas simples sobre seus clientes, principalmente sobre seus padrões de visita, quanto gastaram e quais itens do cardápio são seus favoritos. Ter essa conexão mais profunda com seus clientes o ajudará a oferecer uma experiência melhor e mais personalizada. 
 Ele planeja usar essas informações para ajudá-lo a decidir se deve expandir o programa de fidelidade do cliente existente. Além disso, ele precisa de ajuda para gerar alguns conjuntos de dados básicos para que sua equipe possa inspecionar os dados facilmente sem precisar usar SQL.

# 2. Estrutura dos dados

O dataset é composto por três tabelas, que na verdade são amostras dos dados gerais, e o Diagrama Entidade-Relacionamento está apresentado abaixo.

<img width="400" height="209" alt="image" src="https://github.com/user-attachments/assets/d4b21b84-ee89-4332-b4bb-2697190709ce" />

Tabela 1 - Sales

A tabela "Sales" registra todas as compras realizadas por cliente, identificado qual produto e em que datas os pedidos foram realizados.

| Atributos |	Descrição |
| --------- | --------- |
| customer_id | id do cliente |
| order_date | data do pedido |
| product_id | id do produto |

Tabela 2 - Menu

A tabela "Menu" mapeia todos os produtos do cardápio, contendo seu id, nome e preço.

| Atributos |	Descrição |
| --------- | --------- |
| product_id | id do produto |
| product_name | nome do produto |
| price | preço do produto |

Tabela 3 - Members

A tabela "Members" registra quando um cliente ingressou no programa de fidelidade.
| Atributos |	Descrição |
| --------- | --------- |
| customer_id | id do cliente |
| join_date | data de ingresso no programa |

# 3. Estudos

Nessa etapa, deve-se responder às perguntas feitas pelo proprietário do restaurante.

**1. Qual o valor total gasto por cada cliente no restaurante?**

- Somar o total gasto por cada cliente por meio de um JOIN entre as tabelas sales e menu e agrupar por customer_id.

Query:

    SELECT 
    	customer_id, 
        SUM (price) AS total_gasto
    FROM sales as s
    LEFT JOIN menu as m
    	ON s.product_id = m.product_id
    GROUP BY s.customer_id
    ORDER BY customer_id ASC
    
Resultado:

| customer_id | total_gasto |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |



[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)

**2. Qual o total de visitas feitas ao restaurante por cada cliente?**

- Utilizar COUNT DISTINCT para contar quantos dias cada cliente foi ao restaurante. É importante usar a palavra-chave DISTINCT ao calcular a contagem de visitas para evitar a contagem duplicada de dias. Por exemplo, se o Cliente A visitou o restaurante duas vezes em '2021-01-07', a contagem sem DISTINCT resultaria em 2 dias em vez da contagem correta de 1 dia. Ainda, se o cliente pediu mais de um produto no mesmo dia, a contagem dos dias de visita também seria duplicada sem o uso do DISTINCT.
  
Query:

    SELECT 
    	customer_id, 
        COUNT (DISTINCT order_date)
    FROM sales
    GROUP BY customer_id
    ORDER BY customer_id

Resultado:

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

O cliente B é o que mais frequentou o restaurante até então.

**3. Qual foi o primeiro produto do cardápio comprado por cada cliente?**

- Criar uma tabela temporária por meio de uma CTE para criar um ranking dos produtos comprados por cada cliente a partir da data em que o pedido foi feito.
- Depois, fazer uma consulta dentro dessa CTE buscando o produto onde o rank é igual a 1 e agrupando por cliente, retornando, assim, o primeiro produto comprado por cada cliente.

Query:

    WITH ordered_sales_cte AS (
      SELECT 
        sales.customer_id, 
        sales.order_date, 
        menu.product_name,
        DENSE_RANK() OVER (
          PARTITION BY sales.customer_id 
          ORDER BY sales.order_date) AS rank
      FROM dannys_diner.sales
      INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
    )
    
    SELECT 
      customer_id, 
      product_name
    FROM ordered_sales_cte
    WHERE rank = 1
    GROUP BY customer_id, product_name;

Resultado:

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

O primeiro pedido do cliente A incluiu um sushi e um curry.
O primeiro pedido do cliente B foi um curry.
O primeiro pedido do cliente C foi um ramen.

**4. Qual é o item mais pedido pelos clientes e quantas vezes foi pedido?**

- Fazer uma contagem com o uso do COUNT de todos os itens pedidos, ordenar por ordem decrescente em relação à frequência de pedidos de cada item com uso do ORDER BY.
- Utilizar a cláusula LIMIT 1 para filtrar os resultados, trazendo apenas o item mais pedido.

    SELECT
    	menu.product_name,
    	COUNT (sales.product_id) as produto_mais_pedido
    FROM sales
    LEFT JOIN menu
    ON sales.product_id = menu.product_id
    GROUP BY menu.product_name
    ORDER BY produto_mais_pedido DESC
    LIMIT 1

Resultado:

| product_name | produto_mais_pedido |
| ------------ | ------------------- |
| ramen        | 8                   |

O ramen foi pedido 8 vezes e é o campeão de vendas.

**5. Qual é o item mais popular de cada cliente?**

- Criar uma CTE de nome contagem_produto_pedido_cte que conte quantas vezes cada item foi pedido por cada cliente.
- Utilzar a função DENSE_rANK() para fazer um ranking dos itens pedidos por cada cliente a partir da quantidade de vezes que o item foi pedido, em ordem decrescente. Assim, o item mais pedido é o item número 1 no ranking.
- Na consulta externa, aplicar a cláusula WHERE para retornar apenas os itens que possuem rank = 1.

Query:

    WITH contagem_produto_pedido_cte AS (
    SELECT
        sales.customer_id,
        menu.product_name,
        COUNT(sales.product_id) AS frequencia_pedido,
        DENSE_RANK() OVER (
            PARTITION BY sales.customer_id
            ORDER BY COUNT(sales.product_id) DESC
        ) AS rank
    FROM sales
    LEFT JOIN menu
        ON sales.product_id = menu.product_id
    GROUP BY sales.customer_id, menu.product_name
    ORDER BY sales.customer_id, rank
    )
     
    SELECT 
    	customer_id,
        product_name
    FROM contagem_produto_pedido_cte
    WHERE rank = 1
    GROUP BY customer_id, product_name
    ORDER BY customer_id

Resultado:

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | curry        |
| B           | ramen        |
| B           | sushi        |
| C           | ramen        |

Clientes A e C preferem ramen.
Cliente B gosta de todo o cardárpio.

**6. Qual item foi comprado primeiro pelo cliente depois que ele se tornou membro?**

- Criar uma CTE com o nome pedidos_como_membro para trazer os pedidos realizados após o cliente se tornar membro.
- Utilizar a função ROW_NUMBER() com a cláusula PARTITION BY para particionar os dados pelo customer_id e ordená-los pela data do pedido em ordem crescente.
- Fazer um INNER JOIN entre as tabelas sales e members e selecionar apenas as vendas onde order_date for maior que join_date, ou seja, apenas as vendas realizadas depois que o cliente se tornou membro.
-  Na consulta externa, fazer um LEFT JOIN entre a CTE pedidos_como_membro e a tabela menu para trazer o customer_id e o product_name.
-  Utilizar a cláusula WHERE para filtrar as linhas trazendo apenas as que possuem row_num = 1, que representa a primeira linha dentro do particionamento do customer_id.

Query:

    WITH pedidos_como_membro AS (
      SELECT
      	sales.customer_id,
        sales.product_id,
        ROW_NUMBER() OVER (
          PARTITION BY members.customer_id
          ORDER BY sales.order_date
        ) AS row_num
       FROM sales
       INNER JOIN members
       ON sales.customer_id = members.customer_id
       AND sales.order_date > members.join_date
        )
        
    SELECT
    	pedidos_como_membro.customer_id,
        menu.product_name
    FROM pedidos_como_membro
    LEFT JOIN menu
    ON pedidos_como_membro.product_id = menu.product_id
    WHERE row_num = 1
    ORDER BY pedidos_como_membro.customer_id

Resultado:

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |

Após virarem membros, o primeiro pedido do cliente A foi um ramen e do cliente B foi um sushi.

**7. Qual item foi comprado logo antes do cliente se tornar membro?**

- Basicamente o mesmo processo da questão anterior, porém dessa vez dentro da CTE, o INNER JOIN é feito seguindo a condição de que deve trazer apenas as vendas realizadas antes do cliente se tornar membro, portanto sales.order_date < members.join_date. Ainda, a função ROW_NUMBER() deve usar ORDER BY sales.order_date DESC, para trazer as datas dos pedidos em ordem decrescente.
- Com essas alterações, a query externa permanece a mesma, pois agora row_num = 1 trará a última venda feita antes do cliente se tornar membro.

Query:

    WITH pedidos_antes_membro AS (
      SELECT
      	sales.customer_id,
        sales.product_id,
        ROW_NUMBER() OVER (
          PARTITION BY members.customer_id
          ORDER BY sales.order_date DESC
        ) AS row_num
       FROM sales
       INNER JOIN members
       ON sales.customer_id = members.customer_id
       AND sales.order_date < members.join_date
        )
        
    SELECT
    	pedidos_como_membro.customer_id,
        menu.product_name
    FROM pedidos_como_membro
    LEFT JOIN menu
    ON pedidos_como_membro.product_id = menu.product_id
    WHERE row_num = 1
    ORDER BY pedidos_como_membro.customer_id

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |

Tanto cliente A quanto cliente B pediram sushi logo antes de se tornarem membros.

**8. Qual item foi comprado logo antes do cliente se tornar membro?**

- Utilizar a mesma CTE da questão anterior, que traz as vendas realizadas antes do cliente se tornar membro.
- Na query externa, com a função COUNT() contar quantos itens cada cliente pediu e com a função SUM() somar o total gasto.
- Fazer um INNER JOIN entre a CTE pedidos_antes_membro e a tabela menu, para poder obter as informações referentes ao preço de cada item.
- Com a cláusular GROUP BY, agrupar por customer_id.

Query:

    WITH pedidos_antes_membro AS (
          SELECT
          	sales.customer_id,
            sales.product_id
           FROM sales
           INNER JOIN members
           ON sales.customer_id = members.customer_id
           AND sales.order_date < members.join_date
    )
    
    SELECT
    	p.customer_id,
       	COUNT(p.product_id) AS total_amount,
      	SUM(m.price) AS amount_spent
    FROM pedidos_antes_membro AS p
    INNER JOIN menu AS m
    	ON p.product_id = m.product_id
    GROUP BY p.customer_id
    ORDER BY p.customer_id

Resultado:

| customer_id | total_amount | amount_spent |
| ----------- | ------------ | ------------ |
| A           | 2            | 25           |
| B           | 3            | 40           |

Cliente A pediu 2 itens e gastou $25 dólares antes de se tornar membro.
Cliente B pediu 3 itens e gastou $40 dólares antes de se tornar membro.

**9. Se cada dólar gasto equivale a 10 pontos e o sushi tem um multiplicador de pontos de 2x, quantos pontos cada cliente teria??**

- Considerar que a questão envolve todos os clientes e todo o período.
- 
