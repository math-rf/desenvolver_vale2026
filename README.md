# Guia de Configuração: Trabalhando com Notebooks (Modo Híbrido: PY + MD)

Este repositório utiliza o **Jupytext** para gerenciar arquivos de Jupyter Notebook de forma avançada. Para evitar conflitos de merge e garantir a melhor visualização possível, nós **versionamos apenas os arquivos `.py` e `.md**` no Git. O arquivo original `.ipynb` fica ignorado localmente.

### Por que essa abordagem híbrida?

* **`.py` (Formato Percent):** Contém apenas o código puro e limpo. É o arquivo perfeito para fazer *Code Review* de lógica e resolver merges difíceis.
* **`.md` (Markdown):** Contém o código e os **gráficos gerados**. Perfeito para visualizar os resultados finais diretamente pelo navegador no GitHub/GitLab sem precisar baixar nada.

---

## 🚀 Passo a Passo para quem deu Git Pull

### 1. Ative seu ambiente virtual e instale o Jupytext

Abra o terminal da sua IDE e execute:

```bash
pip install jupytext

```

### 2. Recrie o arquivo `.ipynb` a partir dos arquivos baixados

Para gerar o notebook interativo local na sua máquina, execute o comando de sincronização apontando para o script Python:

```bash
jupytext --sync nome_do_seu_arquivo.py

```

*(O Jupytext lerá o `.py` e o `.md` e recriará o `.ipynb` perfeitamente na sua máquina).*

### 3. Abra e trabalhe normalmente

Pronto! O arquivo `.ipynb` aparecerá na sua árvore de arquivos. Abra-o e execute as células normalmente. Sempre que você salvar (`Ctrl + S`), tanto o `.py` quanto o `.md` serão atualizados sozinhos.

---

## ✨ Criando um Novo Notebook do Zero

Se você precisa criar uma análise ou modelo totalmente novo no projeto, siga estes passos para ativar o Jupytext nele:

1. Crie o arquivo do notebook normalmente na sua IDE (ex: `minha_analise.ipynb`).
2. Adicione algum código inicial e **salve o arquivo** pelo menos uma vez (`Ctrl + S`).
3. Abra o terminal da IDE e execute o comando abaixo para ativar o pareamento triplo:
```bash
jupytext --set-formats ipynb,py:percent,md minha_analise.ipynb

```


4. Verifique se os arquivos `minha_analise.py` e `minha_analise.md` foram criados automaticamente na sua pasta. A partir de agora, eles serão atualizados a cada salvamento do notebook.

---

## 🛠️ Como enviar suas alterações (Git Commit)

O arquivo `.gitignore` está configurado para ignorar arquivos `*.ipynb`. O Git só vai rastrear as modificações do `.py` e do `.md`.

1. Modifique e execute o seu `.ipynb`.
2. **Salve o notebook** (isso força o Jupytext a atualizar os outros dois arquivos).
3. Adicione e envie os arquivos gerados:

```bash
git add nome_do_seu_arquivo.py nome_do_seu_arquivo.md
git commit -m "feat: nova análise de dados e gráficos atualizados"
git push

```

---

## 💡 Como resolver conflitos de merge

Se houver um conflito causado por alterações simultâneas:

1. Abra o arquivo **`.py`** no PyCharm e resolva o conflito de código facilmente (já que é apenas texto de código puro).
2. Se o arquivo `.md` também reclamar de conflito por causa das imagens geradas, limpe as marcações de conflito do Git no `.md` (ou aceite as alterações de uma das ramificações).
3. No terminal, force a sincronização a partir do seu `.py` corrigido:
```bash
jupytext --sync nome_do_seu_arquivo.py

```


4. Abra o `.ipynb`, execute as células novamente para gerar os gráficos finais e salve para que o `.md` seja reescrito corretamente de forma automática.