Atividade de Banco de Dados - Jefferson Ribeiro Dos Santos

##  1. Total de usuários cadastrados
```sql
SELECT COUNT(*) FROM auth.users;
```

##  2. Peso médio dos usuários
```sql
SELECT AVG(peso) AS peso_medio
FROM usuarios;
```

##  3. Usuário com maior meta diária
```sql
SELECT nome, email, meta_diaria_ml
FROM usuarios
ORDER BY meta_diaria_ml DESC
LIMIT 1;
```

##  4. Usuário com menor meta diária
```sql
SELECT nome, email, meta_diaria_ml
FROM usuarios
ORDER BY meta_diaria_ml ASC
LIMIT 1;
```

##  5. Total de água ingerida por usuário
```sql
SELECT 
    u.id,
    u.nome,
    SUM(i.quantidade_ml) AS total_ingerido_ml
FROM usuarios u
LEFT JOIN ingestao_agua i 
    ON u.id = i.usuario_id
GROUP BY u.id, u.nome
ORDER BY total_ingerido_ml DESC NULLS LAST;
```

##  6. Total de água em um dia específico
```sql
SELECT SUM(quantidade_ml) AS total_dia_ml
FROM ingestao_agua
WHERE data_hora >= '2026-04-15 00:00:00'
  AND data_hora < '2026-04-16 00:00:00';
```

##  7. Usuário que mais bebeu no período
```sql
SELECT 
    u.id,
    u.nome,
    SUM(i.quantidade_ml) AS total_ml
FROM usuarios u
JOIN ingestao_agua i 
    ON u.id = i.usuario_id
WHERE i.data_hora BETWEEN '2026-04-01' AND '2026-04-30'
GROUP BY u.id, u.nome
ORDER BY total_ml DESC
LIMIT 1;
```

##  8. Usuário que menos bebeu no período
```sql
SELECT 
    u.id,
    u.nome,
    COALESCE(SUM(i.quantidade_ml), 0) AS total_ml
FROM usuarios u
LEFT JOIN ingestao_agua i 
    ON u.id = i.usuario_id
    AND i.data_hora BETWEEN '2026-04-01' AND '2026-04-30'
GROUP BY u.id, u.nome
ORDER BY total_ml ASC
LIMIT 1;
```

##  9. Consumo médio diário por usuário
```sql
SELECT 
    u.id,
    u.nome,
    AVG(d.total_ml) AS media_diaria_ml
FROM usuarios u
LEFT JOIN (
    SELECT 
        usuario_id,
        DATE(data_hora) AS dia,
        SUM(quantidade_ml) AS total_ml
    FROM ingestao_agua
    GROUP BY usuario_id, DATE(data_hora)
) d ON u.id = d.usuario_id
GROUP BY u.id, u.nome
ORDER BY media_diaria_ml DESC;
```

##  10. Média por ingestão (copo)
```sql
SELECT AVG(quantidade_ml) AS media_ml_por_copo
FROM ingestao_agua;
```

##  11. Consumo diário por usuário
```sql
SELECT 
    u.nome,
    DATE(i.data_hora) AS dia,
    SUM(i.quantidade_ml) AS total_ml_dia
FROM usuarios u
JOIN ingestao_agua i 
    ON u.id = i.usuario_id
GROUP BY u.nome, DATE(i.data_hora)
ORDER BY u.nome, dia;
```

##  12. Dia com maior consumo total
```sql
SELECT 
    DATE(data_hora) AS dia,
    SUM(quantidade_ml) AS total_ml
FROM ingestao_agua
GROUP BY DATE(data_hora)
ORDER BY total_ml DESC
LIMIT 1;
```

##  13. Dia com menor consumo total
```sql
SELECT 
    DATE(data_hora) AS dia,
    SUM(quantidade_ml) AS total_ml
FROM ingestao_agua
GROUP BY DATE(data_hora)
ORDER BY total_ml ASC
LIMIT 1;
```

##  14. Quantidade de registros por dia
```sql
SELECT 
    DATE(data_hora) AS dia,
    COUNT(*) AS total_registros
FROM ingestao_agua
GROUP BY DATE(data_hora)
ORDER BY dia;
```

##  15. Usuários que atingiram a meta por dia
```sql
SELECT 
    u.nome,
    DATE(i.data_hora) AS dia,
    SUM(i.quantidade_ml) AS total_ml_dia,
    u.meta_diaria_ml
FROM usuarios u
JOIN ingestao_agua i 
    ON u.id = i.usuario_id
GROUP BY u.id, u.nome, DATE(i.data_hora), u.meta_diaria_ml
HAVING SUM(i.quantidade_ml) >= u.meta_diaria_ml
ORDER BY dia, u.nome;
```

##  16. Dias que cada usuário bateu a meta
```sql
SELECT 
    u.nome,
    COUNT(*) AS dias_que_bateu_meta
FROM usuarios u
JOIN (
    SELECT 
        usuario_id,
        DATE(data_hora) AS dia,
        SUM(quantidade_ml) AS total_ml
    FROM ingestao_agua
    GROUP BY usuario_id, DATE(data_hora)
) d ON u.id = d.usuario_id
WHERE d.total_ml >= u.meta_diaria_ml
GROUP BY u.nome
ORDER BY dias_que_bateu_meta DESC;
```

##  17. Percentual da meta atingida por dia
```sql
SELECT 
    u.nome,
    DATE(i.data_hora) AS dia,
    SUM(i.quantidade_ml) AS total_ml_dia,
    u.meta_diaria_ml,
    (SUM(i.quantidade_ml) * 100.0 / u.meta_diaria_ml) AS percentual_meta
FROM usuarios u
JOIN ingestao_agua i 
    ON u.id = i.usuario_id
GROUP BY u.id, u.nome, DATE(i.data_hora), u.meta_diaria_ml
ORDER BY u.nome, dia;
```

##  18. Melhor desempenho (mais dias batendo meta)
```sql
SELECT 
    u.nome,
    COUNT(*) AS dias_com_meta_atingida
FROM usuarios u
JOIN (
    SELECT 
        usuario_id,
        DATE(data_hora) AS dia,
        SUM(quantidade_ml) AS total_ml
    FROM ingestao_agua
    GROUP BY usuario_id, DATE(data_hora)
) d ON u.id = d.usuario_id
WHERE d.total_ml >= u.meta_diaria_ml
GROUP BY u.nome
ORDER BY dias_com_meta_atingida DESC
LIMIT 1;
```

##  19. Horário de maior consumo
```sql
SELECT 
    EXTRACT(HOUR FROM data_hora) AS hora,
    SUM(quantidade_ml) AS total_ml
FROM ingestao_agua
GROUP BY EXTRACT(HOUR FROM data_hora)
ORDER BY total_ml DESC
LIMIT 1;
```

##  20. Intervalo médio entre ingestões
```sql
SELECT 
    usuario_id,
    AVG(intervalo_minutos) AS intervalo_medio_min
FROM (
    SELECT 
        usuario_id,
        EXTRACT(EPOCH FROM (data_hora - LAG(data_hora) OVER (
            PARTITION BY usuario_id 
            ORDER BY data_hora
        ))) / 60 AS intervalo_minutos
    FROM ingestao_agua
) t
WHERE intervalo_minutos IS NOT NULL
GROUP BY usuario_id
ORDER BY intervalo_medio_min;
```
