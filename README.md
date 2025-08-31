# Análise do IPCA Mensal

## 📊 Objetivo
Analisar a variação mensal do índice oficial de preços do Brasil, e suas composições.  

**Geral**: Índice Geral  
**Grupos**: 
   1. Alimentação e Bebidas
   2. Habitação
   3. Artigos de Residência
   4. Vestuário
   5. Transportes
   6. Saúde e Cuidados Pessoais
   7. Despesas Pessoais
   8. Educação
   9. Comunicação 

## Passo a Passo da Análise

### 1. Extração de Dados do SIDRA
Extração de dados do IPCA Mensal - Tabela 7060 do Sistema IBGE de Recuperação Automática (SIDRA). 
```r
# Extraindo os dados com SidraR
library(sidrar)
dados <- get_sidra(api = '/t/7060/n1/all/v/63/p/last%2013/c315/7169,7170,7445,7486,7558,7625,7660,7712,7766,7786/d/v63%202')
```

### 2. Preparação dos Dados
```r
# Tratando os dados
ipca <- dados |> 
  select(grupo = `Geral, grupo, subgrupo, item e subitem`,
         mes = Mês,
         med_cod = `Mês (Código)`,
         valor = Valor) |> 
  mutate(data = parse_date(med_cod,
                           format = '%Y%m'))
```

### 3. Visualização Gráfica
Gráficos dos Grupos/Itens:
```r
ggplot(ipca, aes(x = data , y = valor), size = 15) +
  geom_line(aes(group = grupo), linewidth = 1.2, color = '#F97D24') +
  facet_wrap(~grupo, ncol = 2, scales = 'free') +
  labs(title = 'IPCA: Taxa de Inflação e Componentes',
       subtitle = 'Variação Mensal (%)',
       x = NULL,
       y = '(%)',
       caption = 'Fonte: IBGE | Elaboração: Caio Nunes') +
  scale_x_date(date_breaks = '3 month',
               date_labels = '%b %Y') +
  theme(panel.background = element_rect(fill = 'white'),
        plot.title = element_text(family = 'dosis', hjust = 0.5, face = 'bold', color = 'black', size = 19),
        plot.subtitle = element_text(family = 'poppins', hjust = 0.5),
        plot.caption = element_text(family = 'poppins', hjust = 1, vjust = 0,.8, color = 'black', size = 9),
        legend.position = 'none',
        legend.text = element_blank(),
        legend.title = element_blank(),
        axis.title = element_text(family = 'poppins', hjust = 0.5, color = 'black', size = 9),
        axis.text = element_text(family = 'poppins', hjust = 0.5, color = 'black', size = 9),
        axis.line.x = element_line(),
        axis.line.y = element_line())
```
