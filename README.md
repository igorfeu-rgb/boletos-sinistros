# Boletos → CSV (Extrator em Python)

Extrai informações de **boletos em PDF** e gera um **CSV em UTF‑8** com as colunas:

- `Linha digitável ou Código de barras*`
- `CNPJ/CPF*` (o script prioriza **CNPJ do FIDC/Beneficiário**; nunca usa CPF do cliente)
- `Data de agendamento*`
- `Descrição*` (nome do arquivo PDF)
- `Tags`

> Dependência principal: [`pdfplumber`](https://pypi.org/project/pdfplumber/).

---

## ✨ Funcionalidades

- Leitura de PDFs com `pdfplumber` (texto extraível; PDFs somente-imagem exigem OCR).
- Detecção da **linha digitável/código de barras** (formato pontuado ou 47–48 dígitos).
- Extração de **CNPJ do FIDC/Beneficiário** por heurísticas de proximidade a âncoras como `fidc`, `beneficiário`, `cedente`, `sacador/avalista` etc.
- **Data de agendamento** configurável (padrão: **hoje**).
- **Descrição** definida automaticamente como o **nome do arquivo PDF**.
- Geração de CSV em UTF‑8 (sem BOM), criando a pasta de saída se não existir.
- Mensagens de aviso em `stderr` para arquivos problemáticos, sem interromper o processamento dos demais.

---

## 🧰 Requisitos

- **Python 3.8+**
- **pdfplumber**
  ```bash
  pip install pdfplumber
  ```
- Sistema operacional: testado no **Windows**, compatível com Linux/macOS (ajuste caminhos).

> Dica: use um ambiente virtual (opcional)
> ```bash
> python -m venv .venv
> .venv\Scripts\activate   # Windows
> # source .venv/bin/activate  # Linux/macOS
> pip install pdfplumber
> ```

---

## ⚙️ Configuração

Edite o bloco `CONFIG` no início do script:

```python
CONFIG = {
    "INPUT_DIR": r"C:\Users\Igor Feu\Downloads\Boletos Iugu",
    "OUTPUT_CSV": r"C:\Users\Igor Feu\Downloads\Boletos Iugu\boletos.csv",
    "DESCRICAO": None,  # ignorado (Descrição será o nome do arquivo PDF)
    "TAGS": "Pagamento, Despesa, Cliente",
    # Data de agendamento sempre o dia de execução:
    "AGENDAMENTO_SOURCE": "today",
    "AGENDAMENTO_DATE": None,  # não usado com "today"
}
```

Parâmetros:

- `INPUT_DIR`: pasta onde estão os PDFs dos boletos.
- `OUTPUT_CSV`: caminho do CSV de saída (a pasta é criada se não existir).
- `DESCRICAO`: ignorado (o script usa o nome do arquivo).
- `TAGS`: string livre que será repetida para todas as linhas.
- `AGENDAMENTO_SOURCE` (opções):
  - `"today"` → usa a data de hoje.
  - `"fixed"` → usa a data fixa em `AGENDAMENTO_DATE` (**formato `DD/MM/AAAA`**).
  - `"filedate"` → pega a data do **arquivo** (timestamp de modificação).
  - `"vencimento"` → tenta inferir pelo rótulo "vencimento" no PDF.
- `AGENDAMENTO_DATE`: obrigatório se `AGENDAMENTO_SOURCE="fixed"` (formato `DD/MM/AAAA`).

---

## ▶️ Como usar

1. Coloque os PDFs dos boletos dentro de `INPUT_DIR`.
2. Execute o script:
   ```bash
   python seu_arquivo.py
   ```
3. Ao final, você verá algo como:
   ```
   OK: 12 registro(s) gravado(s) em C:\Users\Igor Feu\Downloads\Boletos Iugu\boletos.csv
   ```

O CSV terá o cabeçalho:
```csv
Linha digitável ou Código de barras*,CNPJ/CPF*,Data de agendamento*,Descrição*,Tags
```

Exemplo de linha (fictícia):
```csv
34191.79001 01043.510047 91020.150008 1 92360000012345,12.345.678/0001-90,19/06/2025,BOLETO123.pdf,"Pagamento, Despesa, Cliente"
```

---

## 🔎 Como a extração funciona (resumo técnico)

- **Linha digitável / código de barras**
  - Procura primeiro próximo ao rótulo "linha digitável".
  - Em seguida, busca no texto inteiro por formato **pontuado** (ex.: `XXXXX.XXXXX XXXXX.XXXXXX ...`) ou por **47–48 dígitos**.

- **CNPJ (FIDC/Beneficiário)**
  - Heurística de prioridade por proximidade às âncoras (em ordem):  
    `fidc` → `beneficiário/beneficiario` → `cedente` → `sacador/avalista` → primeiro CNPJ do documento.
  - **Nunca retorna CPF**: se não encontrar CNPJ, o campo fica vazio.

- **Data de agendamento**
  - Depende de `AGENDAMENTO_SOURCE` (vide seção de configuração). Padrão: `"today"`.

- **Descrição**
  - Nome do arquivo PDF (`os.path.basename(path)`).

---

## 🧪 Limitações e observações

- PDFs **escaneados** (apenas imagem) **não têm texto** para o `pdfplumber`. Use OCR antes (ex.: [OCRmyPDF](https://ocrmypdf.readthedocs.io/) com Tesseract) para torná-los pesquisáveis.
- Layouts de boleto muito fora do padrão podem exigir ajuste nas **âncoras** ou nas **janelas de busca**.
- Se `INPUT_DIR` não existir, o script encerra com erro.
- Em caso de falha ao extrair de um arquivo, o script **registra aviso** em `stderr` e continua os demais.

---

## 🛠️ Personalizações úteis

- **Novas âncoras** para CNPJ: adicione termos no array `anchors` da função `choose_cnpj_fidc`.
- **Estratégia de data**: altere `AGENDAMENTO_SOURCE` para `"fixed"`, `"filedate"` ou `"vencimento"`.
- **Tags por arquivo**: adapte para ler tags de um mapeamento por nome de arquivo, se necessário.
- **CLI/Parâmetros**: você pode substituir a `CONFIG` por argumentos de linha de comando (ex.: `argparse`).

---

## 🧯 Solução de problemas

- **`Erro: instale a dependência com pip install pdfplumber`**  
  Instale a lib: `pip install pdfplumber`. Em ambientes corporativos, verifique proxy/repositório interno.

- **`INPUT_DIR inválido`**  
  Ajuste o caminho em `CONFIG["INPUT_DIR"]` e verifique se a pasta existe.

- **`Nenhum texto extraído de ...`**  
  O PDF provavelmente é imagem. Rode OCR e tente novamente.

- **Saída com campos vazios**  
  Pode indicar variação no layout do boleto. Considere ajustar âncoras ou ampliar janelas de busca na função de extração.

---

## 📂 Estrutura sugerida do projeto

```
boletos-sinistros/
├─ src/
│  └─ extrator_boletos.py
├─ data/
│  └─ input/              # PDFs
├─ output/
│  └─ boletos.csv
├─ README.md
└─ requirements.txt
```

**requirements.txt** (exemplo):
```
pdfplumber>=0.11
```

---

## 📄 Licença

Defina a licença que preferir (ex.: MIT). Exemplo de cabeçalho no `LICENSE`:
```
MIT License — © 2025 Igor Feu
```

---

## 🙌 Créditos

Script e README por **Igor Feu** (com apoio do ChatGPT). Sinta‑se à vontade para abrir *issues* e melhorias.
