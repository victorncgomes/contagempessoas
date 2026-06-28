# Prompt para a Extensão Claude (Chrome) — Subir ao GitHub e abrir no Colab

## Como usar
1. Abra o **Claude para Chrome**.
2. Tenha em mãos os arquivos baixados: `Sistema_Hibrido_Contagem_Pessoas_Colab.ipynb`,
   `RELATORIO_Alteracoes.md` e o original `Modelo_YOLOv11.ipynb`.
3. Substitua o que estiver entre «aspas angulares» pelos seus dados.
4. Cole o prompt abaixo na extensão.

> **Importante (limite real):** a extensão é ótima para abrir páginas, subir arquivos e iniciar a
> execução. Ela **não** deve ficar de babá durante o treino do CSRNet (pode levar horas). O notebook
> salva checkpoint a cada época no seu Drive e **retoma sozinho** — então, se a sessão cair, basta
> reabrir e rodar de novo. Use a extensão para **montar e disparar**, não para vigiar o treino.

---

## PROMPT (copie a partir daqui)

```
Você vai me ajudar a publicar um projeto no GitHub e abri-lo no Google Colab. Siga os passos na ordem
e me peça confirmação antes de qualquer ação destrutiva (apagar/sobrescrever).

CONTEXTO
- É um projeto de pós-graduação: um sistema híbrido de contagem de pessoas (YOLOv11 + CSRNet).
- Arquivos no meu computador: "Sistema_Hibrido_Contagem_Pessoas_Colab.ipynb",
  "RELATORIO_Alteracoes.md" e o arquivo original "Modelo_YOLOv11.ipynb".

PARTE A — GITHUB
1. Abra «URL_DO_SEU_REPOSITORIO».
   (Se eu ainda NÃO tenho repositório: abra https://github.com/new e crie um repositório
    chamado "contagem-multidao-hibrido", público, com um README inicial.)
2. Crie esta estrutura de pastas/arquivos no repositório (use "Add file" → "Upload files"):
   - na raiz: "Sistema_Hibrido_Contagem_Pessoas_Colab.ipynb"
   - na raiz: "RELATORIO_Alteracoes.md"
   - crie a pasta "original/" e suba ali o "Modelo_YOLOv11.ipynb"
     (mantemos o arquivo original do colega para dar crédito e mostrar a evolução do projeto).
3. Edite o README.md com este conteúdo:
   - Título: "Sistema Híbrido de Contagem de Pessoas (YOLOv11 + CSRNet)"
   - Um parágrafo explicando que o projeto cobre cenas esparsas (YOLO) e densas (CSRNet),
     com um roteador que escolhe o modelo automaticamente.
   - Uma seção "Como rodar" dizendo: abrir o notebook da raiz no Google Colab, ativar a GPU T4,
     e executar as células na ordem.
   - Uma seção "Créditos" citando que a base de detecção (Modelo A) partiu do notebook original
     do colega, preservado em original/Modelo_YOLOv11.ipynb.
   - Uma linha apontando para o RELATORIO_Alteracoes.md.
4. Faça o commit das alterações. Me mostre a URL final do repositório.

PARTE B — GOOGLE COLAB
5. Abra https://colab.research.google.com
6. Abra o notebook "Sistema_Hibrido_Contagem_Pessoas_Colab.ipynb":
   - opção 1: aba "GitHub", cole a URL do repositório e selecione o notebook; ou
   - opção 2: "Fazer upload" e selecione o arquivo do meu computador.
7. Vá em "Ambiente de execução" → "Alterar o tipo de ambiente de execução" → selecione "T4 GPU" → salvar.
8. Execute as células da seção 0 (Setup) e confirme que aparece o nome da GPU (T4).
9. PARE aqui e me avise. O treino do CSRNet é demorado; eu decido quando disparar.
   Lembre-se: o treino salva checkpoint a cada época e retoma sozinho se cair — não fique vigiando.

REGRAS
- Antes de sobrescrever qualquer arquivo existente, me pergunte.
- Ao final de cada parte, me dê um resumo curto do que foi feito e os links.
```

(fim do prompt)

---

## Dica
A Parte A (mexer no repositório/Git) costuma ser mais suave no **Claude Code desktop**, que você já tem
— ele lida com `git` diretamente. Se preferir, use o Claude Code para criar o repo e subir os arquivos,
e reserve a extensão do Chrome para a Parte B (Colab). Funciona dos dois jeitos; é questão de gosto.
