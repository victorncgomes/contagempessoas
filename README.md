# Sistema Híbrido de Contagem de Pessoas (YOLOv11 + CSRNet)

Projeto de pós-graduação que combina dois modelos de visão computacional para contar pessoas em qualquer tipo de cena. Em cenas esparsas (uma sala, uma rua, um evento pequeno), o YOLOv11 detecta cada pessoa individualmente e conta as caixas. Em multidões densas (estádios, shows, manifestações), o CSRNet estima um mapa de densidade e obtém a contagem pela integral desse mapa. Um roteador automático executa o YOLO primeiro e, com base na contagem e no tamanho médio das caixas detectadas, decide sozinho se a cena é densa o bastante para escalar para o CSRNet — sem o usuário precisar escolher o modelo manualmente.

## Como rodar

Abra o notebook da raiz, `Sistema_Hibrido_Contagem_Pessoas_Colab.ipynb`, no Google Colab. Em seguida vá em **Ambiente de execução → Alterar o tipo de ambiente de execução** e selecione **T4 GPU**. Por fim, execute as células na ordem, começando pela seção 0 (Setup). O treino do CSRNet salva um checkpoint a cada época no Google Drive e retoma de onde parou caso a sessão caia.

## Créditos

A base de detecção (Modelo A) partiu do notebook original de um colega, preservado neste repositório em `original/Modelo_YOLOv11.ipynb`. Esse núcleo de detecção foi mantido, corrigido e estendido ao longo do projeto, e os demais componentes (CSRNet e o roteador híbrido) foram construídos sobre essa fundação.

## Alterações

O detalhamento das mudanças em relação ao notebook original está em RELATORIO_Alteracoes.md.
