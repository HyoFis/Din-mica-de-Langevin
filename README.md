# Semana da Física --- Dinâmica de Langevin

Este notebook investiga numericamente a dinâmica de uma partícula em um
potencial de poço duplo, submetida a uma força aleatória. O estudo
compara diferentes intensidades de ruído e observa seus efeitos sobre a
energia mecânica e sobre os tempos de primeira passagem entre os poços.

## Objetivos

-   Visualizar o potencial de poço duplo utilizado na simulação.
-   Implementar a evolução determinística e a dinâmica de Langevin com
    ruído.
-   Calcular a energia cinética, a energia potencial e a energia
    mecânica média em várias realizações.
-   Registrar o tempo de primeira passagem da partícula entre as regiões
    (x `\leq 0`{=tex}) e (x \> 0).
-   Construir a contagem acumulada de primeiras passagens e comparar os
    resultados para diferentes intensidades de ruído.

## Modelo físico

O potencial utilizado é

\[ V(x)=4x^4-4x^2. \]

Ele possui dois mínimos em

\[ x\_{`\min`{=tex}}=`\pm`{=tex}`\frac{1}{\sqrt{2}}`{=tex},
`\qquad `{=tex}V(x\_{`\min`{=tex}})=-1, \]

e uma barreira em (x=0), onde (V(0)=0).

A força conservativa é obtida por

\[ -`\frac{dV}{dx}`{=tex}=-(16x\^3-8x). \]

Na função `passo_ruido`, a atualização do momento inclui a força do
potencial, o termo de atrito e uma contribuição aleatória gaussiana:

\[ p\_{n+1}=p_n-`\Delta `{=tex}t(16x_n\^3-8x_n)
-`\beta `{=tex}p_n`\Delta `{=tex}t+F`\sqrt{\Delta t}`{=tex},`\eta`{=tex}\_n,
\]

em que (`\eta`{=tex}\_n) é uma variável aleatória normal padrão. Em
seguida, a posição é atualizada por

\[ x\_{n+1}=x_n+`\Delta `{=tex}t,p\_{n+1}. \]

Essa é a regra numérica implementada no notebook; a interpretação física
e a convenção de discretização dependem das unidades e da definição do
modelo adotado.

## Parâmetros da simulação

Os valores definidos no notebook são:

  ----------------------------------------------------------------------------------------
  Parâmetro                                                    Valor Descrição
  ------------------------- ---------------------------------------- ---------------------
  `dt`                                                         0,001 Passo temporal

  `beta`                                                           1 Coeficiente de atrito

  `exp`                                                          500 Número de realizações
                                                                     por intensidade

  `passos`                                                    20.000 Passos temporais por
                                                                     realização

  `intensidade_aleatoria`            `[1.5, 1.25, 0.99, 0.68, 0.42]` Intensidades do ruído

  `lista_porcentagem`         `["100%", "75%", "50%", "25%", "10%"]` Rótulos usados no
                                                                     gráfico de energia
  ----------------------------------------------------------------------------------------

O tempo total simulado por realização é de aproximadamente 20 unidades
de tempo, pois `passos * dt = 20`.

A condição inicial da posição é

\[ x(0)=-`\frac{1}{\sqrt{2}}`{=tex}, \]

correspondente ao mínimo esquerdo do potencial. O momento inicial é
zero, pois os vetores de momento são inicializados com zeros.

## Organização do notebook

### 1. Importações

São utilizadas as bibliotecas:

-   `numpy`: operações numéricas e geração de números aleatórios;
-   `matplotlib.pyplot`: construção dos gráficos;
-   `tqdm`: barra de progresso das simulações;
-   `scipy.optimize.curve_fit`: ajuste da curva acumulada.

### 2. Potencial de estudo

Calcula e plota (V(x)=4x^4-4x^2), destacando os dois mínimos e a
barreira central em (x=0).

### 3. Funções da dinâmica

-   `passo(x, p)`: atualiza posição e momento sem o termo de ruído e sem
    atrito.
-   `passo_ruido(x, p)`: atualiza a dinâmica incluindo atrito e força
    aleatória.

A função `passo_ruido` utiliza as variáveis globais `dt`, `beta` e `F`.
Portanto, `F` deve estar definido com a intensidade correspondente antes
de chamar a função.

### 4. Parâmetros experimentais

Define as intensidades do ruído, o passo temporal, o atrito, o número de
experimentos e a quantidade de passos.

### 5. Energia mecânica média

Para cada intensidade de ruído, são realizadas 500 trajetórias. Em cada
passo são calculadas:

\[ K(t)=`\frac{p(t)^2}{2}`{=tex}, `\qquad`{=tex} V(t)=4x(t)^4-4x(t)^2,
\]

e a energia mecânica:

\[ E(t)=K(t)+V(t). \]

As grandezas são somadas entre as realizações e divididas pelo número de
experimentos, produzindo médias temporais. O notebook também calcula a
média da energia mecânica após o índice 5000 e seu desvio padrão, usados
para representar uma linha de referência e uma faixa no gráfico.

### 6. Tempo de primeira passagem

Cada trajetória começa no poço esquerdo. O espaço é dividido em duas
regiões:

-   região esquerda: (x`\leq 0`{=tex});
-   região direita: (x\>0).

O código identifica a primeira mudança da região esquerda para a direita
e registra o tempo correspondente em `tempo_pp`. O indicador `passou`
impede que cruzamentos posteriores da mesma trajetória sejam registrados
novamente.

Assim, `tempo_pp[i, exps]` guarda o tempo da primeira passagem da
realização `exps` para a intensidade de ruído `i`.

Trajetórias que não cruzam a fronteira durante o intervalo simulado
permanecem com valor zero no vetor `tempo_pp`, pois a matriz é
inicializada com zeros.

### 7. Distribuição acumulada das primeiras passagens

Para cada intensidade, o notebook conta quantos tempos registrados são
positivos e menores que cada instante de referência. O resultado é
`PP_time`, que representa o número acumulado de primeiras passagens
observadas até aquele instante.

A curva é ajustada pela função

\[ N(t)=a-b,e\^{-ct}, \]

em que `a`, `b` e `c` são parâmetros estimados por `curve_fit`. O
parâmetro `c` controla a escala de crescimento do ajuste; não deve ser
interpretado diretamente como uma probabilidade.

### 8. Tempos individuais por intensidade

O último gráfico apresenta os tempos de primeira passagem de cada
realização em função da intensidade do ruído. Cada ponto corresponde a
uma trajetória simulada. Valores iguais a zero indicam que não foi
registrada uma primeira passagem durante o tempo de simulação.

## Como executar

1.  Abra o arquivo `Semana_da_Física,_Dinâmica_de_Langevin.ipynb` em
    Jupyter Notebook, JupyterLab ou Google Colab.
2.  Certifique-se de que as bibliotecas listadas em **Importações**
    estejam instaladas.
3.  Execute as células em ordem, do início ao fim.
4.  Observe os gráficos gerados para comparar a energia mecânica e as
    primeiras passagens nas diferentes intensidades de ruído.

Instalação das dependências, caso necessário:

``` bash
pip install numpy matplotlib tqdm scipy notebook
```

## Cuidados e pontos de verificação

Antes de utilizar os resultados em uma análise quantitativa ou
publicação, recomenda-se verificar os seguintes pontos do código:

1.  **Inicialização de `tempo_pp`:** a célula de parâmetros utiliza
    `len(forcas)`, mas `forcas` não está definido nas células
    apresentadas. Se não for definido em outra parte do ambiente,
    substitua por:

    ``` python
    tempo_pp = np.zeros((len(intensidade_aleatoria), exp))
    ```

2.  **Escala de tempo no gráfico acumulado:** `tempo_pp` armazena tempos
    em unidades físicas (`contador * dt`), enquanto o gráfico usa
    `np.arange(101)` e compara com inteiros. Para manter as unidades
    consistentes, defina explicitamente o vetor de tempos de referência
    e compare-o com os tempos registrados.

3.  **Condição de primeira passagem:** o código registra a primeira
    mudança de região. Como a condição inicial está no poço esquerdo,
    isso corresponde à primeira passagem para (x\>0). O incremento do
    contador ocorre no passo em que a mudança é detectada.

4.  **Trajetórias sem passagem:** os zeros em `tempo_pp` representam
    ausência de passagem observada até o fim da simulação, e não uma
    passagem ocorrida em tempo zero. Ao calcular frações acumuladas ou
    ajustar distribuições de tempos, considere essas trajetórias não
    observadas (censuradas).

5.  **Ajuste exponencial:** verifique a qualidade do ajuste e a
    interpretação dos parâmetros. A função ajustada é uma aproximação
    empírica da contagem acumulada.

6.  **Reprodutibilidade:** para reproduzir uma execução, pode-se definir
    uma semente antes das simulações, por exemplo `np.random.seed(123)`.
    Sem uma semente fixa, os resultados variam entre execuções.

7.  **Variável global `F`:** como `passo_ruido` consulta `F`
    globalmente, confirme que ela foi atualizada para a intensidade
    correta antes de cada conjunto de trajetórias.

## Saídas produzidas

O notebook gera três tipos principais de visualização:

1.  **Evolução temporal da energia mecânica:** compara a energia
    mecânica média para as intensidades de ruído estudadas.
2.  **Distribuição acumulada dos tempos de primeira passagem:** mostra o
    número de trajetórias que já realizaram a primeira passagem em
    função do tempo, junto aos ajustes exponenciais.
3.  **Tempos individuais de primeira passagem:** mostra a dispersão dos
    tempos registrados para cada intensidade de ruído.

## Resumo

O notebook simula uma partícula em um potencial de poço duplo sob
dinâmica de Langevin. Variando a intensidade do ruído, investiga-se como
a energia mecânica evolui e como muda a ocorrência de transições do poço
esquerdo para o direito. A energia média, a contagem acumulada e os
tempos individuais de primeira passagem fornecem perspectivas
complementares sobre o comportamento estocástico do sistema.
