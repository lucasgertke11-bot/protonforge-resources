# MakaiForger Resources

Repositório central de metadados do Makai Forger. Contém apenas um arquivo — `metadata.json` — que informa ao app quais versões de cada resource estão disponíveis para download.

## O que faz

Quando o Makai Forger inicia, ele:

1. Baixa este `metadata.json` (tenta GitHub primeiro, fallback GitLab)
2. Compara a versão local vs remota de cada resource
3. Se `localVersion < remoteVersion`, baixa a versão nova

## Estrutura

```
metadata.json
```

### Schema do metadata.json

```json
{
  "version": 1781120306,
  "resources": {
    "catalogo.db.gz": {
      "version": 1,
      "tag": "v0.0.1",
      "sha256": "af72a2fd992a5a6274035137dd5421f029683c61e50d0d5936bf7fc264a7555e",
      "size": 55951414,
      "original_size": 263843840,
      "github_repo": "MakaiForger/makaiforger-catalogo-db",
      "gitlab_project_id": 83081380
    }
  }
}
```

| Campo | Descrição |
|-------|-----------|
| `version` | Timestamp Unix da publicação (sempre incrementa) |
| `resources.*.version` | Número da versão do resource (1, 2, 3...) |
| `resources.*.tag` | Tag do release no GitHub (ex: `v0.0.2`) |
| `resources.*.sha256` | SHA256 do arquivo compactado |
| `resources.*.size` | Tamanho em bytes do arquivo compactado |
| `resources.*.original_size` | Tamanho descomprimido (0 para .tar.gz) |
| `resources.*.github_repo` | Repositório GitHub do resource |
| `resources.*.gitlab_project_id` | Project ID no GitLab (fallback) |

### Resources atuais (6)

| Resource | Repo GitHub | Tamanho | Conteúdo |
|----------|-------------|---------|----------|
| `catalogo.db.gz` | makaiforger-catalogo-db | 53 MB | Catálogo de jogos |
| `proton_data.db.gz` | makaiforger-proton-data-db | 74 MB | Dados de Proton |
| `fork_catalog.db.gz` | makaiforger-fork-catalog-data | 34 KB | Catálogo de forks |
| `game_dlls.db.gz` | makaiforger-game-dlls-data | 2.9 KB | Mapa DLLs x jogos |
| `releases.tar.gz` | makaiforger-releases-data | 303 KB | Releases antigos |
| `installer-api.tar.gz` | makaiforger-installer-api | 6.6 KB | API de instalação |

## Como o app usa

No `resource-manager.ts`:

```
App inicia
  │
  ├── 1. Baixa metadata.json de:
  │     https://github.com/MakaiForger/makaiforger-resources/raw/main/metadata.json
  │     (fallback: GitLab project 83110951)
  │
  ├── 2. Para cada resource em KNOWN_RESOURCES:
  │     ├── Versão local < remota? → baixa
  │     ├── SHA256 confere? → extrai
  │     └── Falhou? → loga e continua
  │
  └── 3. Salva metadata.json local em userData/resources/metadata.json
```

## Quando publicar uma atualização

Sempre que um dos 6 resources for atualizado (ex: novo fork adicionado no fork_catalog.db), faça:

### 1. Publique o resource no repo dele

```bash
# Exemplo: fork_catalog.db.gz
gzip -kf resources/fork_catalog.db

# Crie release v0.0.2 no GitHub MakaiForger/makaiforger-fork-catalog-data
# Asset: fork_catalog.db.gz

# Upload GitLab fallback
curl -X PUT "https://gitlab.com/api/v4/projects/83110945/packages/generic/makaiforger-fork-catalog-data/v0.0.2/fork_catalog.db.gz" \
  -H "PRIVATE-TOKEN: $GL_TOKEN" \
  -H "Content-Type: application/gzip" \
  --data-binary @fork_catalog.db.gz
```

### 2. Atualize este metadata.json

Edite `resources/metadata.json` no projeto principal, depois commite e push aqui:

```bash
# Calcule o SHA256 e size do novo arquivo
sha256sum resources/fork_catalog.db.gz

# Atualize a entrada no metadata.json:
#   "version": 2,
#   "tag": "v0.0.2",
#   "sha256": "<novo_sha>",
#   "size": <novo_tamanho>,

# Push
git add metadata.json
git commit -m "atualiza fork_catalog.db.gz para v0.0.2"
git push
```

### 3. Faça o mesmo no GitLab

```bash
curl -X PUT "https://gitlab.com/api/v4/projects/83110951/repository/files/metadata.json" \
  -H "PRIVATE-TOKEN: $GL_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"branch\":\"main\",\"content\":$(python3 -c "import json; print(json.dumps(open('resources/metadata.json').read()))"),\"commit_message\":\"atualiza fork_catalog.db.gz para v0.0.2\"}"
```

Ou use o script automatizado:

```bash
GH_TOKEN=... GL_TOKEN=... npx tsx scripts/publish-resources.ts
```

O script publica cada resource + atualiza este metadata.json automaticamente.

## Fallback

| Prioridade | Origem | URL |
|------------|--------|-----|
| 1ª | GitHub | `https://github.com/MakaiForger/makaiforger-resources/raw/main/metadata.json` |
| 2ª | GitLab | `https://gitlab.com/api/v4/projects/83110951/repository/files/metadata.json/raw?ref=main` |

## Versionamento

Não usa tags. O `metadata.json` sempre na `main`. A versão global é o timestamp no campo `version`.

Projeto mantido por **desenvolvedor** independente.
