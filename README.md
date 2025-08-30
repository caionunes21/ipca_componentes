# Análise do IPCA Mensal

## 📊 Objetivo
Analisar a variação mensal do índice oficial de preços do Brasil, e suas composições.

## Dataset
- **Variável Dependente:** Produtividade
- **Variáveis Independentes:** 
  - Fertilizantes (quantitativa)
  - Qualidade do Solo (quantitativa)
  - Lavoura (quantitativa)
- Fonte: Dados fictícios

## Passo a Passo da Análise

### 1. Extração de Dados do SIDRA
```r
# Extraindo os dados com SidraR
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

### 3. Interpretação do Modelo
Principais saídas do `summary(modelo)`:
- **Coeficientes:** Efeito de cada variável na produtividade
    - Coeficiente Angular (β0): -16.43062
    - Coeficiente Fertilizantes (β1): 0.82955
    - Coeficiente Qualidade do Solo (β2): 4.99885
    - Coeficiente Lavoura (β3): 0.19941
- **R-squared:** 0.9592 (Proporção da variabilidade explicada)
- **p-valores:** Significância estatística (p < 0.05 = significativo)
    - Coeficiente Angular: 7.55e-05 ***
    - Coeficiente Fertilizantes: 1.05e-06 ***
    - Coeficiente Qualidade do Solo: 0.0112 ***
    - Coeficiente Lavoura: 0.19941 *

### 4. Visualização Gráfica
Gráficos da Variação Mensal:
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

### 5. Pressupostos da Regressão
Verifique sempre:
1. Linearidade (gráfico resíduos vs ajustados)
```r
  ggplot(data = NULL, aes(x = fitted(modelo), y = residuals(modelo))) +
  geom_point(size = 2, colour = 'black') +
  geom_hline(yintercept = 0, linetype = "dashed") +
  labs(x = "Valores Ajustados", y = "Resíduos", 
       title = "Resíduos X Valores Ajustados") +
  theme(panel.background = element_rect(fill = 'white'),
        plot.title = element_text(family = 'dosis', colour = '#01A77F', hjust = 0.5, size = 14 , face = 'bold'),
        axis.title = element_text(family = 'poppins', hjust = 0.5, color = 'black'),
        axis.text = element_text(family = 'poppins', hjust = 0.5, color = 'black'),
        axis.line.x = element_line(),
        axis.line.y = element_line())
```
2. Homocedasticidade (Teste White)
```r
white(modelo, interactions = TRUE)
```
**Resultado:** Válido supor que há homocedasticidade
- Hipótese Nula (HO): Homocedasticidade
- X²: 12.4 com 9 graus de liberdade
- P-valor: (0.192 > 0.05)



3. Normalidade dos resíduos (Shapiro-Wilk)
4. Multicolinearidade (VIF)

## Resultados Esperados
- Equação de regressão:  
  `Produtividade = β0 + β1*Fertilizantes + β2*Qualidade_solo + β3*Lavoura`
- Relação quantitativa entre insumos agrícolas e produtividade
- Identificação de fatores mais impactantes
