# Geonode

##### O repositório contém uma instância do Geonode para FUNCEME. Abaixo segue a explicação de como foi feita construída a instância. Caso queira apenas levantar o contêiner, siga para 

##### [Criação do Arquivo de Configuração de Ambiente (dotEnv)](#criação-do-arquivo-de-configuração-de-ambiente-dotenv)

##### e em seguida 

##### [Subindo o Contêiner pela primeira vez](#subindo-o-contêiner-pela-primeira-vez)


# Etapas de Criação da Instância GeoNode do Zero - geoFunceme

## Instalação

### Preparação

#### Criação do Ambiente Virtual e Instalação do Django

```bash
python3 -m venv ~/.venvs/project_name
source ~/.venvs/geofunceme/bin/activate

pip install Django==3.2.16
```

#### Template Geonode

```bash
git clone https://github.com/GeoNode/geonode-project.git -b 4.2.2
```

#### Criação da Instância

```bash
django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile geofunceme
```

#### Criação do Arquivo de Configuração de Ambiente (dotEnv)

##### Navegue para o diretório da instância

```bash
cd /opt/geonode_custom/geofunceme

python3 create-envfile.py \
  --hostname geo.funceme.br \
  --email jefferson.galvao@funceme.br \
```
O `create-envfile.py` aceita os seguintes argumentos:

- `--https`: Habilita SSL. Por padrão está desabilitado.
- `--env_type`:
  - Quando definido como `prod`, o `DEBUG` é desabilitado e a criação de um SSL válido é solicitada ao servidor ACME da Letsencrypt.
  - Quando definido como `test`, o `DEBUG` é desabilitado e um certificado SSL de teste é gerado para testes locais.
  - Quando definido como `dev`, o `DEBUG` é habilitado e nenhum certificado SSL é gerado.
- `--hostname`: A URL que servirá o GeoNode (localhost por padrão).
- `--email`: O e-mail do administrador. Observe que um e-mail real e configurações SMTP válidas são necessárias se `--env_type` estiver configurado para `prod`. A Letsencrypt usa o e-mail para emitir o certificado SSL.
- `--geonodepwd`: Senha do administrador do GeoNode. Um valor aleatório é definido se deixado vazio.
- `--geoserverpwd`: Senha do administrador do GeoNode. Um valor aleatório é definido se deixado vazio.
- `--pgpwd`: Senha do administrador do PostgreSQL. Um valor aleatório é definido se deixado vazio.
- `--dbpwd`: Senha do usuário do banco de dados do GeoNode. Um valor aleatório é definido se deixado vazio.
- `--geodbpwd`: Senha do usuário do banco de dados de dados do GeoNode. Um valor aleatório é definido se deixado vazio.
- `--clientid`: ID do cliente do cliente OAuth2 do GeoServer no GeoNode. Um valor aleatório é definido se deixado vazio.
- `--clientsecret`: Segredo do cliente do cliente OAuth2 do GeoServer no GeoNode. Um valor aleatório é definido se deixado vazio.

### Etapas Adicionais

#### Tradução para Português do Brasil

Para aplicar o idioma (que não vem por padrão no GeoNode) é preciso seguir alguns passos:

##### Criar pasta `locale`

```bash
cd geonode_custom/geofunceme/src/geofunceme
mkdir locale
```

#### Inserir arquivos .po e .mo para o idioma pt_br

Copiar os diretórios e arquivos contidos em `traducao/locale/*` para a pasta `locale`
```bash
cp -r ./geofunceme/customizacao_geonode/traducao/locale/* ./geofunceme/src/geofunceme/locale/
```
#### Editar dotEnv
```bash
nano .env
```
Localizar a linha comentada `# LANGUAGES=(('en-us','English'), [...]`
e substituir por `LANGUAGES=(('en-us','English'),('it-it','Italiano'), ('pt-br', 'Português (Brasileiro)'), ('fr', 'Français'))`
Aqui constará os idiomas disponíveis na plataforma

#### Editar o `settings.py`
```bash
cd geonode_custom/geofunceme/src/geofunceme
nano settings.py
```
Localize se há a linha:
```python
LANGUAGE_CODE = os.getenv("LANGUAGE_CODE", "en")
LANGUAGES = [
    ('en', 'English'),
    [...]
]
```
Copie (ou substitua as linhas supracitadas) no `settings.py` a linha abaixo:
```python
LANGUAGE_CODE = os.getenv("LANGUAGE_CODE", "en")
LANGUAGES = [
    ('en', 'English'),
    ('pt-br', 'Português (Brasil)'),
    ('fr', 'Français'),
    ('it', 'Italiano')
]
```
### Customização dos templates GeoNode

#### Mover Estáticos para a pasta
```bash
cp ./geofunceme/customizacao_geonode/templates_modificados/img/* ./geofunceme/src/geofunceme/static/img
```
#### Mover templates snippets para a pasta
```bash
cp -r ./geofunceme/customizacao_geonode/templates_modificados/geonode-mapstore-client/* ./geofunceme/src/funceme_mapas/templates/geonode-mapstore-client
```

### Subindo o Contêiner pela primeira vez

Aqui precisa de persistência e paciência, pois é comum dar falhas ao construir a imagem, pela grande quantidade de requisitos.

#### Acesse a pasta da instância

```bash
cd ./geofunceme
```

#### Execute o docker build

```bash
docker compose build
```

#### Levante o container

```bash
docker compose up -d
```

#### Em casos de falha ao levantar o container

Reconstrua as imagens ignorando o cache

```bash
docker compose build --no-cache
```

Tente levantar novamente o serviço
```bash
docker compose up -d
```

**Repita até funcionar**

### Informações adicionais

Diretório contendo as telas front-end no container django:
```bash
cd /usr/local/lib/python3.10/dist-packages/geonode_mapstore_client/templates/geonode-mapstore-client/snippets/
```
## Autores

- [Jefferson Sant'ana Galvão](https://gitlab-ce.com/jefferson.galvao) - Geógrafo, Analista de Sistemas. Especialista em Sistema de Informação Espacial (Geoprocessamento).