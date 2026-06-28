# Relatório Técnico — Da Base do Colega ao Sistema Híbrido de Contagem

**Projeto:** Contagem de pessoas em imagem (pós-graduação)
**Arquivo original:** `Modelo_YOLOv11.ipynb` (autoria do colega)
**Arquivo final:** `Sistema_Hibrido_Contagem_Pessoas_Colab.ipynb`
**Objetivo deste documento:** explicar, de forma detalhada e honesta, o que existia, quais
problemas foram identificados, o que foi corrigido, por que a arquitetura mudou e como a base
construída pelo colega permaneceu sendo o núcleo do sistema.

---

## 1. Sumário executivo

O notebook original era um **contador de pessoas por detecção**: carregava o YOLOv11 pré-treinado,
rodava em uma imagem fixa, contava as caixas da classe "person" e exibia a imagem anotada.
Funcionava — detectou 6 pessoas no teste do colega — mas tinha limitações técnicas pontuais
(um bug de cor visível, caminho fixo de Windows, ausência de tratamento de erro) e, sobretudo,
**não atendia à exigência do trabalho**: o enunciado pede um modelo que possa ser **treinado
(fine-tuning)** e cuja **precisão seja avaliada**. Detecção com pesos prontos não treina nada.

Além disso, há um limite conceitual: **YOLO não conta multidão densa**. Para cobrir as duas
situações exigidas (cenas esparsas *e* densas), o projeto evoluiu para uma **arquitetura híbrida**:
dois modelos especialistas (YOLO + CSRNet) e um **roteador** que escolhe qual usar conforme a cena.

A base do colega não foi descartada — ela é o **Modelo A** do sistema final, apenas corrigida,
encapsulada em função e integrada ao conjunto maior.

---

## 2. O que o colega construiu (a fundação)

O original tinha 8 células com um fluxo limpo e didático:

| Célula | Conteúdo |
|---|---|
| 0 (texto) | Título e os 5 passos do processo |
| 1 (código) | Instalação das dependências |
| 2 (código) | Importações + carregamento do `yolo11n.pt` |
| 3 (texto) | Cabeçalho "Informe o caminho da imagem" |
| 4 (código) | Caminho da imagem a analisar |
| 5 (código) | Detecção + contagem das pessoas (classe 0 do COCO) |
| 6 (código) | Contagem alternativa pelo nome da classe ("person") |
| 7 (código) | Geração e exibição da imagem anotada |

**Mérito da base:** o colega montou corretamente o *pipeline* essencial de detecção —
carregar o modelo, rodar a inferência, percorrer as detecções, filtrar a classe "person",
contar e visualizar. Esse encadeamento é exatamente o coração de um contador por detecção e é o
que permanece vivo no sistema final. Toda a parte de multidão densa e o roteador foram construídos
**por cima** dessa fundação, não no lugar dela.

---

## 3. Problemas identificados no arquivo original

Em ordem do mais concreto ao mais conceitual:

**3.1. Bug de cor (BGR→RGB) — célula 7.**
`resultado.plot()` devolve a imagem no padrão **BGR** (convenção do OpenCV), mas o `matplotlib`
espera **RGB**. Sem inverter os canais, as cores saem trocadas e a pele das pessoas fica azulada.
É um erro *visível* na saída do notebook — funciona, conta certo, mas entrega a imagem com aparência
errada. Em um trabalho avaliado, isso chama atenção negativa.

**3.2. Caminho absoluto de Windows — célula 4.**
`"C:/Users/hamota/Desktop/Trabalho/Pessoas03.jpg"` só existe na máquina do colega. O notebook não
roda no Google Colab nem na máquina de quem for avaliar.

**3.3. Ausência de tratamento de erro.**
Se o caminho estiver errado ou nenhuma pessoa for detectada, o código quebra com erro técnico
em vez de uma mensagem clara.

**3.4. Contagem duplicada — células 5 e 6.**
As duas células contam a mesma coisa (uma pelo índice da classe, outra pelo nome). Provavelmente
sobra da fase de estudo; em um produto final é redundante.

**3.5. Confiança fixa no meio do código — célula 5.**
O limiar `conf=0.5` está cravado dentro da chamada. Para ajustar e justificar a escolha, deveria ser
um parâmetro visível.

**3.6. Instalação redundante — célula 1.**
No Colab, `opencv`, `matplotlib` e `pillow` já vêm instalados; só a Ultralytics é necessária.

**3.7. Sem prontidão para Colab/GPU.**
Nenhuma menção a ambiente de execução, GPU ou upload de imagem.

**3.8. Detalhes cosméticos.**
Parêntese solto em `# Carrega o modelo)` (célula 2) e `#Informe o caminho da imagem` sem espaço após
o `#` (célula 3), que o Markdown interpreta como título grande.

**3.9. (Conceitual) Escopo insuficiente para o trabalho.**
O notebook só faz **inferência** com um modelo pronto. O enunciado exige **treinar (fine-tuning)** e
**avaliar a precisão**. Detecção com pesos congelados não cumpre isso.

**3.10. (Conceitual) Limite de método para multidão — ver seção 4.**

---

## 4. A problemática do YOLO com multidões

Este é o ponto técnico mais importante do projeto, e precisa estar claro no relatório.

"Contar pessoas" são, na prática, **dois problemas diferentes**:

- **Detecção** (o que o YOLO faz): desenha uma caixa por pessoa e conta as caixas. Funciona enquanto
  é possível **distinguir um indivíduo do outro** — sala, calçada, fila, evento médio.
- **Estimativa de densidade** (CSRNet e similares): **não detecta ninguém individualmente**; prevê um
  *mapa de densidade* (mapa de calor) e estima o total pela **integral** desse mapa.

Em uma **multidão densa** (estádio, show, manifestação), as cabeças viram pontos minúsculos e
sobrepostos. A detecção falha por oclusão: o YOLO simplesmente **não enxerga** cada pessoa e
**subconta** de forma grave. E — ponto crucial — **nenhum fine-tuning resolve isso**, porque o
problema não é falta de treino: é a abordagem (detecção) que é inadequada para o caso denso.

**Conclusão:** para cobrir *cenas esparsas e densas*, não existe um único modelo YOLO que dê conta.
São paradigmas distintos, com métricas distintas (mAP na detecção; MAE/RMSE na densidade).

---

## 5. A decisão pela arquitetura híbrida

Como o trabalho precisa atender às **duas situações**, a solução profissional é um **sistema híbrido**:
dois modelos especialistas + um **roteador** que decide qual usar.

- **Modelo A — YOLOv11** (base do colega): cenas esparsas. Detecta e conta.
- **Modelo B — CSRNet**: cenas densas. Estima densidade e conta pela integral. É aqui que entra o
  **fine-tuning** exigido (ver seção 6).
- **Roteador**: escolhe o modelo conforme a cena.

**O problema do ovo e da galinha.** Para "escolher pelo número de pessoas", seria preciso já saber o
número — que é o que queremos descobrir. A saída explora uma assimetria conhecida: **o YOLO é confiável
exatamente quando a cena é esparsa.** Então:

1. Roda-se o **YOLO primeiro** (é rápido).
2. Se ele detecta **pouca gente** → confia-se nele; a cena é esparsa.
3. Se ele detecta **muita gente** (ou caixas minúsculas, indicando cabeças pequenas de multidão) →
   escala-se para o **CSRNet**.

Dois sinais baratos sustentam a decisão: a **contagem** do YOLO e o **tamanho médio das caixas**
(pessoas grandes = perto = esparso; caixas minúsculas = multidão). Os limiares são **parâmetros
calibráveis** no topo do notebook.

**Honestidade técnica (vale ponto no relatório):** o limiar pode errar na **fronteira** (cenas de
~100–150 pessoas), roteando para o modelo errado. Isso não é defeito a esconder: é uma limitação a
**apontar e calibrar** usando o próprio estudo comparativo entre os dois modelos. A integração não
dispensa a comparação — ela **depende** dela.

---

## 6. O que foi feito — correções e construção

### 6.1. Correções herdadas da base (Modelo A)
| Problema original | Correção aplicada |
|---|---|
| Bug BGR→RGB (3.1) | `resultado.plot()[:, :, ::-1]` converte para RGB antes de exibir |
| Caminho de Windows (3.2) | Entrada por upload (`files.upload()`) com imagem de exemplo de *fallback* |
| Sem tratamento de erro (3.3) | Fluxo tolerante: usa exemplo se não houver upload; funções encapsuladas |
| Contagem duplicada (3.4) | Mantida apenas a contagem por nome ("person"), mais legível |
| `conf` fixo (3.5) | Detecção encapsulada em função com `conf` ajustável |
| Instalação redundante (3.6) | Só `ultralytics` |
| Sem GPU/Colab (3.7) | Checagem de GPU + instruções de runtime + Drive para checkpoints |
| Cosméticos (3.8) | Reescritos |

### 6.2. Construção nova (Modelo B — CSRNet, com fine-tuning)
- Implementação **moderna do CSRNet** em PyTorch (front-end VGG16 pré-treinada no ImageNet +
  back-end de convoluções dilatadas). Como a VGG16 já vem treinada, o treino é **fine-tuning** legítimo.
- **Dataset ShanghaiTech** (Parte A densa / Parte B esparsa) baixado do Kaggle com **cache no Drive**.
- Geração do **ground truth** por kernel gaussiano sobre os pontos de cabeça.
- **Avaliação por MAE/RMSE**, com **baseline antes** do treino e medição **depois** (tabela antes×depois).
- **Treino com checkpoint por época + retomada automática** (sobrevive à queda da GPU grátis) e
  **curvas de aprendizado** (loss e MAE por época) — exatamente o "ir treinando e melhorando" exigido.

### 6.3. Construção nova (Roteador híbrido)
- Função de decisão `eh_denso()` baseada em contagem do YOLO + tamanho médio das caixas, com limiares
  parametrizados.
- Função `contar()` que orquestra: roda o YOLO, decide o regime e devolve a contagem do modelo correto,
  com a visualização apropriada (caixas no esparso; mapa de calor no denso).

---

## 7. Comparação detalhada: antes × depois

| Aspecto | Original (`Modelo_YOLOv11`) | Final (`Sistema_Hibrido`) |
|---|---|---|
| Modelos | 1 (YOLO, inferência) | 2 (YOLO + CSRNet) + roteador |
| Treino / fine-tuning | Nenhum | CSRNet com fine-tuning da VGG16 |
| Avaliação de precisão | Nenhuma | MAE/RMSE, baseline e antes×depois |
| Curvas de aprendizado | — | Loss e MAE por época |
| Cobre cena densa | Não (subconta) | Sim (CSRNet) |
| Cobre cena esparsa | Sim | Sim (YOLO) |
| Cor da imagem | Bug (azulada) | Corrigida (RGB) |
| Entrada da imagem | Caminho fixo Windows | Upload + exemplo |
| Parâmetros | Cravados no código | Bloco de parâmetros no topo |
| Robustez a queda da GPU | — | Checkpoint + retomada |
| Roda no Colab | Não | Sim (com GPU) |
| Estrutura | 8 células lineares | 20 células em 5 seções |

---

## 8. Como a base do colega permaneceu

O núcleo de detecção escrito pelo colega **não foi substituído** — foi promovido a **Modelo A** do
sistema. O encadeamento original (carregar YOLO → inferir → filtrar "person" → contar → exibir anotada)
está integralmente preservado dentro da função `detectar_yolo()`. As únicas mudanças nesse núcleo são:
a correção do bug de cor, o uso da contagem por nome e o empacotamento em função reutilizável (que
também devolve o tamanho médio das caixas, usado pelo roteador). Em resumo: **a ideia, a lógica e o
fluxo são do colega**; o sistema apenas os corrigiu e os colocou para trabalhar junto de um segundo
modelo especialista.

---

## 9. Verificação (duplo-check executado)

Antes de liberar o arquivo, cada peça crítica foi **testada em código**, não apenas revisada por leitura:

- **Sintaxe:** todas as células de código compiladas sem erro.
- **Arquitetura CSRNet:** *forward* validado — entrada (256×384) produz saída (32×48), ou seja, 1/8 da
  resolução, como o CSRNet exige.
- **Mapa de densidade:** a integral do mapa preserva a contagem (4 pontos → soma 4,00).
- **Casamento de resolução:** o alvo reduzido a 1/8 e multiplicado por 64 mantém a contagem (soma 4,00),
  garantindo que o alvo case com a saída do modelo.
- **Parser do `.mat`:** testado contra um arquivo sintético no formato exato do ShanghaiTech.
- **Roteador:** a regra de decisão foi testada em 5 casos (poucas pessoas com caixas grandes → esparso;
  muitas pessoas → denso; contagem média com caixas minúsculas → denso; etc.), todos corretos.

**Transparência:** o duplo-check **encontrou e corrigiu um bug real** — três docstrings haviam sido
geradas com aspas escapadas de forma inválida, o que quebraria duas células no Colab. Foram corrigidas
e o arquivo foi revalidado por completo. Esse é o propósito do duplo-check: pegar o que a leitura não pega.

---

## 10. Limitações honestas

- O que **não** foi testado aqui é o treino ponta a ponta com dados reais (exige GPU e o dataset, que
  rodam no Colab). O ponto a observar no primeiro run é o nome exato da pasta extraída pelo Kaggle —
  por isso a localização da pasta é feita com busca flexível (`part_A` ou `part_A_final`).
- O ground truth usa kernel gaussiano **fixo** (σ=15). Para a Parte A (densa), o paper original do
  CSRNet usa kernel *geometria-adaptativo*, que melhora um pouco o MAE — fica como upgrade opcional.
- O limiar do roteador deve ser **calibrado** com os próprios testes (ver seção 5).
- Referência de sanidade: implementações públicas de CSRNet ficam em MAE ~68 (Parte A) e ~10 (Parte B);
  o que importa para o trabalho é a **curva de MAE caindo** ao longo das épocas, não bater o número do paper.

---

*Documento gerado para servir de base de explicação ao colega e à banca. Recomenda-se anexá-lo ao
repositório como `RELATORIO_Alteracoes.md`.*
