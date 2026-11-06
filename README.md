# API de Upload e Download de Arquivos

API REST em **Flask** (Python) para enviar (upload) e baixar (download) arquivos, com registro de logs e validações de segurança. Organizada em camadas: rotas, controllers, models e banco de dados (SQLite).

- **Link público (Render):** `https://SEU-SERVICO.onrender.com` *(substitua pelo seu link após o deploy)*
- **Repositório:** `https://github.com/SEU-USUARIO/upload-download-api`

## Tecnologias

- Python 3.13
- Flask 3.1
- SQLite
- Gunicorn (servidor de produção)
- Render (hospedagem)

## Estrutura do projeto

```
upload-download-api/
├── app.py                      # Cria o app Flask e registra as rotas
├── controllers/
│   ├── upload_controller.py    # Lógica de upload (validações, log)
│   └── download_controller.py  # Lógica de download
├── models/                     # Acesso ao banco de dados
├── routes/                     # Definição dos endpoints (Blueprints)
├── database/db.py              # Conexão e criação da tabela SQLite
├── requirements.txt            # Dependências
├── Procfile                    # Comando de start em produção
└── render.yaml                 # Configuração do Render (opcional)
```

## Como executar localmente

```bash
git clone https://github.com/SEU-USUARIO/upload-download-api.git
cd upload-download-api

python -m venv venv
# Windows: venv\Scripts\activate
# Linux/macOS: source venv/bin/activate

pip install -r requirements.txt
python app.py
```

A API sobe em `http://localhost:5000`.

## Endpoints

| Método | Rota                              | Descrição                         |
|--------|-----------------------------------|-----------------------------------|
| GET    | `/`                               | Status da API e lista de rotas    |
| POST   | `/uploads/upload`                 | Envia um arquivo (campo `file`)   |
| GET    | `/downloads/download/<filename>`  | Baixa um arquivo enviado          |

### Upload

- Formato: `multipart/form-data`, campo **`file`**
- Extensões permitidas: `png`, `jpg`, `jpeg`, `gif`, `pdf`, `txt`
- Tamanho máximo: **30 KB**

```bash
curl -X POST -F "file=@exemplo.txt" https://SEU-SERVICO.onrender.com/uploads/upload
```

Resposta (200):
```json
{ "mensagem": "Arquivo salvo com sucesso!", "arquivo": "exemplo.txt" }
```

Erros (400): `Nenhum arquivo enviado`, `Nome do arquivo vazio`, `Arquivo excede o tamanho máximo permitido (30 KB)`, `Tipo de arquivo não permitido!`.

### Download

```bash
curl -OJ https://SEU-SERVICO.onrender.com/downloads/download/exemplo.txt
```

Retorna o arquivo como anexo. Se não existir, retorna **404**.

## Segurança

- `secure_filename` remove caracteres perigosos do nome do arquivo
- Validação de extensão e de tamanho
- Log de uploads (sucesso e falha) com IP em `uploads.log`
- `debug` desligado em produção

## Configuração para produção

- **gunicorn** como servidor (`Procfile`: `web: gunicorn app:app`)
- A porta vem da variável de ambiente `PORT` (definida pelo Render)
- Caminhos de `uploads/`, `database.db` e `uploads.log` absolutos
- `requirements.txt` com versões fixas

## Deploy no Render

1. Crie um **Web Service** em [render.com](https://render.com) e conecte ao repositório do GitHub
2. **Runtime:** Python 3
3. **Build Command:** `pip install -r requirements.txt`
4. **Start Command:** `gunicorn app:app --bind 0.0.0.0:$PORT`
5. **Instance Type:** Free
6. Clique em **Create Web Service** e aguarde o deploy

> **Atenção:** no plano gratuito do Render o disco é efêmero. Arquivos enviados e o `database.db` são apagados a cada novo deploy ou reinício. Além disso, o serviço "dorme" após inatividade, e a primeira requisição pode levar cerca de 1 minuto.
