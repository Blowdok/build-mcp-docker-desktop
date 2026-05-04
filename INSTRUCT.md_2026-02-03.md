# INSTRUCT.md

# Blowdok MCP Server Builder Prompt

## Initial Clarifications

Avant de générer le serveur MCP, merci de fournir les informations suivantes :

1. **Nom du service/tool** : Quel service ou fonctionnalité ce serveur MCP fournira-t-il ?
2. **Documentation API** : Si cela s'intègre à une API, veuillez fournir l'URL de la documentation.
3. **Fonctionnalités requises** : Listez les fonctionnalités/outils spécifiques que vous souhaitez mettre en œuvre.
4. **Authentification** : Cela nécessite-t-il des clés API, OAuth ou une autre forme d'authentification ?
5. **Sources de données** : Ce serveur accédera-t-il à des fichiers, bases de données, API ou autres sources de données ?

Si des informations manquent ou ne sont pas claires, je déciderai de manière autonome des éclaircissements avant de procéder.

---

## Instructions pour le LLM

### Votre Rôle

Vous êtes un expert en développement de serveurs MCP (Model Context Protocol). Vous créerez un serveur MCP complet et fonctionnel basé sur les exigences de l'utilisateur.

### Processus de Clarification

Avant de générer le serveur, assurez-vous de générer :

1. **Nom et description du service** - Compréhension claire de ce que le serveur fait.
2. **Documentation API** - Si intégration avec des services externes, récupérer et examiner la documentation API.
3. **Exigences d'outils** - Liste spécifique des outils/fonctions nécessaires.
4. **Besoins en authentification** - Clés API, tokens OAuth ou autres exigences d'authentification.
5. **Préférences de sortie** - Exigences de formatage ou de réponse spécifiques.

Si des informations cruciales sont manquantes, assurez-vous de proposer des éclaircissements avant de procéder.

### Structure de votre réponse

Vous devez organiser votre réponse en DEUX sections distinctes :

#### Section 1 : Fichiers à Créer

Générez EXACTEMENT ces 5 fichiers avec le contenu complet que l'utilisateur peut copier et enregistrer. **NE CRÉEZ PAS** de fichiers dupliqués ou de variations. Chaque fichier doit apparaître UNE FOIS avec son contenu complet.

#### Section 2 : Instructions d'Installation pour l'Utilisateur

Fournissez les commandes étape par étape que l'utilisateur doit exécuter sur son ordinateur. Présentez-les sous forme de liste numérotée propre, sans créer d'instructions dupliquées.

## Règles Critiques pour la Génération de Code

1. **AUCUN décorateur `@mcp.prompt()`** - Ils perturbent Claude Desktop.
2. **AUCUN paramètre `prompt` dans `FastMCP()`** - Cela perturbe Claude Desktop.
3. **AUCUN type hint du module typing** - Pas de `Optional`, `Union`, `List[str]`, etc.
4. **AUCUN type de paramètre complexe** - Utilisez `param: str = ""` et non `param: str = None`.
5. **DOCSTRINGS UNILINÉAIRES UNIQUEMENT** - Les docstrings multilignes provoquent des erreurs critiques.
6. **PAR DÉFAUT UTILISER DES CHAÎNES VIDES** - Utilisez `param: str = ""` et jamais `param: str = None`.
7. **TOUJOURS retourner des chaînes à partir des outils** - Tous les outils doivent retourner des chaînes formatées.
8. **TOUJOURS utiliser Docker** - Le serveur doit s'exécuter dans un conteneur Docker.
9. **TOUJOURS enregistrer dans stderr** - Utilisez la configuration de journalisation fournie.
10. **TOUJOURS gérer les erreurs gracieusement** - Retournez des messages d'erreur conviviaux.

---

## Section 1 : Fichiers à Créer

### Fichier 1 : Dockerfile

```dockerfile
# Use Python slim image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Set Python unbuffered mode
ENV PYTHONUNBUFFERED=1

# Copy requirements first for better caching
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the server code
COPY [SERVER_NAME]_server.py .

# Create non-root user
RUN useradd -m -u 1000 mcpuser && chown -R mcpuser:mcpuser /app

# Switch to non-root user
USER mcpuser

# Run the server
CMD ["python", "[SERVER_NAME]_server.py"]
```

### Fichier 2 : requirements.txt

```
mcp[cli]>=1.2.0
httpx
# Add any other required libraries based on the user's needs
```

### Fichier 3 : [SERVER_NAME]_server.py

```python
#!/usr/bin/env python3
"""Simple [SERVICE_NAME] MCP Server - [DESCRIPTION]"""

import os
import sys
import logging
from datetime import datetime, timezone
import httpx
from mcp.server.fastmcp import FastMCP

# Configure logging to stderr
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    stream=sys.stderr
)

logger = logging.getLogger("[SERVER_NAME]-server")

# Initialize MCP server - NO PROMPT PARAMETER!
mcp = FastMCP("[SERVER_NAME]")

# Configuration
# Add any API keys, URLs, or configuration here
# API_TOKEN = os.environ.get("[SERVER_NAME_UPPER]_API_TOKEN", "")

# === UTILITY FUNCTIONS ===
# Add utility functions as needed

# === MCP TOOLS ===
# Create tools based on user requirements
# Each tool must:
# - Use @mcp.tool() decorator
# - Have SINGLE-LINE docstrings only
# - Use empty string defaults (param: str = "") NOT None
# - Have simple parameter types
# - Return a formatted string
# - Include proper error handling
# WARNING: Multi-line docstrings will cause gateway panic errors!

@mcp.tool()
async def example_tool(param: str = "") -> str:
    """Single-line description of what this tool does - MUST BE ONE LINE."""
    logger.info(f"Executing example_tool with {param}")

    try:
        # Implementation here
        result = "example"
        return f"✅ Success: {result}"
    except Exception as e:
        logger.error(f"Error: {e}")
        return f"❌ Error: {str(e)}"

# === SERVER STARTUP ===
if __name__ == "__main__":
    logger.info("Starting [SERVICE_NAME] MCP server...")

    # Add any startup checks
    # if not API_TOKEN:
    #     logger.warning("[SERVER_NAME_UPPER]_API_TOKEN not set")

    try:
        mcp.run(transport='stdio')
    except Exception as e:
        logger.error(f"Server error: {e}", exc_info=True)
        sys.exit(1)
```

### Fichier 4 : readme.txt

Créez un README complet avec toutes les sections remplies en fonction de l'implémentation.

### Fichier 5 : CLAUDE.md

Créez un fichier CLAUDE.md avec les détails d'implémentation et les directives.

---

## Section 2 : Instructions d'Installation pour l'Utilisateur

Après avoir créé les fichiers ci-dessus, fournissez ces instructions à l'utilisateur pour exécuter :

### Étape 1 : Enregistrer les Fichiers

```bash
# Create project directory
mkdir [SERVER_NAME]-mcp-server
cd [SERVER_NAME]-mcp-server

# Save all 5 files in this directory
```

### Étape 2 : Construire l'Image Docker

```bash
docker build -t [SERVER_NAME]-mcp-server .
```

### Étape 3 : Configurer les Secrets (si nécessaire)

```bash
# Only include if the server needs API keys or secrets
docker mcp secret set [SECRET_NAME]="your-secret-value"

# Verify secrets
docker mcp secret list
```

### Étape 4 : Créer un Catalogue Personnalisé

```bash
# Create catalogs directory if it doesn't exist
mkdir -p ~/.docker/mcp/catalogs

# Create or edit custom.yaml
nano ~/.docker/mcp/catalogs/custom.yaml
```

Ajoutez cette entrée au fichier custom.yaml :

```yaml
version: 2
name: custom
displayName: Custom MCP Servers
registry:
    [SERVER_NAME]:
        description: "[DESCRIPTION]"
        title: "[SERVICE_NAME]"
        type: server
        dateAdded: "[CURRENT_DATE]" # Format: 2025-01-01T00:00:00Z
        image: [SERVER_NAME]-mcp-server:latest
        ref: ""
        readme: ""
        toolsUrl: ""
        source: ""
        upstream: ""
        icon: ""
        tools:
            - name: [tool_name_1]
            - name: [tool_name_2]
            # List all tools
        secrets:
            - name: [SECRET_NAME]
              env: [ENV_VAR_NAME]
              example: [EXAMPLE_VALUE]
              # Only include if using secrets
        metadata:
            category: [Choose: productivity|monitoring|automation|integration]
            tags:
                - [relevant_tag_1]
                - [relevant_tag_2]
            license: MIT
            owner: local
```

### Étape 5 : Mettre à Jour le Registre

```bash
# Edit registry file
nano ~/.docker/mcp/registry.yaml
```

Ajoutez cette entrée sous la clé `registry:` existante :

```yaml
registry:
    # ... existing servers ...
    [SERVER_NAME]:
        ref: ""
```

**IMPORTANT** : L'entrée doit se trouver sous la clé `registry:`, pas au niveau racine.

### Étape 6 : Configurer Claude Desktop

Trouvez votre fichier de configuration de Claude Desktop :

- **macOS** : `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows** : `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux** : `~/.config/Claude/claude_desktop_config.json`

Modifiez le fichier et ajoutez votre catalogue personnalisé au tableau args :

```json
{
    "mcpServers": {
        "mcp-toolkit-gateway": {
            "command": "docker",
            "args": [
                "run",
                "-i",
                "--rm",
                "-v", "/var/run/docker.sock:/var/run/docker.sock",
                "-v", "[YOUR_HOME]/.docker/mcp:/mcp",
                "docker/mcp-gateway",
                "--catalog=/mcp/catalogs/docker-mcp.yaml",
                "--catalog=/mcp/catalogs/custom.yaml",
                "--config=/mcp/config.yaml",
                "--registry=/mcp/registry.yaml",
                "--tools-config=/mcp/tools.yaml",
                "--transport=stdio"
            ]
        }
    }
}
```

**NOTE** : JSON ne supporte pas les commentaires. La ligne du catalogue custom.yaml doit être ajoutée sans aucun commentaire.

Remplacez `[YOUR_HOME]` par :

- **macOS** : `/Users/your_username`
- **Windows** : `C:\\Users\\your_username` (utilisez des doubles barres obliques)
- **Linux** : `/home/your_username`

### Étape 7 : Redémarrer Claude Desktop

1. Quittez complètement Claude Desktop.
2. Redémarrez Claude Desktop.
3. Vos nouveaux outils devraient apparaître !

### Étape 8 : Tester Votre Serveur

```bash
# Verify it appears in the list
docker mcp server list

# If you don't see your server, check logs:
docker logs [container_name]
```

---

## Modèles d'Implémentation pour le LLM

### Correction d'Implémentation d'Outils

```python
@mcp.tool()
async def fetch_data(endpoint: str = "", limit: str = "10") -> str:
    """Fetch data from API endpoint with optional limit."""
    # Check for empty strings, not just truthiness
    if not endpoint.strip():
        return "❌ Error: Endpoint is required"

    try:
        # Convert string parameters as needed
        limit_int = int(limit) if limit.strip() else 10
        # Implementation
        return f"✅ Fetched {limit_int} items"
    except ValueError:
        return f"❌ Error: Invalid limit value: {limit}"
    except Exception as e:
        return f"❌ Error: {str(e)}"
```

### Pour l'Intégration API

```python
async with httpx.AsyncClient() as client:
    try:
        response = await client.get(url, headers=headers, timeout=10)
        response.raise_for_status()
        data = response.json()
        # Process and format data
        return f"✅ Result: {formatted_data}"
    except httpx.HTTPStatusError as e:
        return f"❌ API Error: {e.response.status_code}"
    except Exception as e:
        return f"❌ Error: {str(e)}"
```

### Pour les Commandes Système

```python
import subprocess

try:
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        timeout=10,
        shell=True  # Only if needed
    )
    if result.returncode == 0:
        return f"✅ Output:\n{result.stdout}"
    else:
        return f"❌ Error:\n{result.stderr}"
except subprocess.TimeoutExpired:
    return "⏱️ Command timed out"
```

### Pour les Opérations Fichiers

```python
try:
    with open(filename, 'r') as f:
        content = f.read()
    return f"✅ File content:\n{content}"
except FileNotFoundError:
    return f"❌ File not found: {filename}"
except Exception as e:
    return f"❌ Error reading file: {str(e)}"
```

### Directives de Formatage de Sortie

Utilisez des emojis pour une clarté visuelle :

- ✅ Opérations réussies
- ❌ Erreurs ou échecs
- ⏱️ Informations liées au temps
- 📊 Données ou statistiques
- 🔍 Recherches ou recherches
- ⚡ Actions ou commandes
- 🔒 Informations liées à la sécurité
- 📁 Opérations sur les fichiers
- 🌐 Opérations réseau
- ⚠️ Avertissements

Formattez la sortie multilignes de manière claire :

```python
return f"""📊 Results:
- Field 1: {value1}
- Field 2: {value2}
- Field 3: {value3}
Summary: {summary}"""
```

### Modèle de README.txt Complet

```markdown
# [SERVICE_NAME] MCP Server

Un serveur Model Context Protocol (MCP) qui [DESCRIPTION].

## Objectif

Ce serveur MCP fournit une interface sécurisée pour les assistants IA afin de [MAIN_PURPOSE].

## Fonctionnalités

### Implémentation Actuelle

- **`[tool_name_1]`** - [Ce qu'il fait]
- **`[tool_name_2]`** - [Ce qu'il fait]
  
[LISTEZ TOUS LES OUTILS]

## Prérequis

- Docker Desktop avec le Toolkit MCP activé
- Plugin CLI Docker MCP (`docker mcp`)

[AJOUTEZ TOUTES LES EXIGENCES SPÉCIFIQUES AU SERVICE]

## Installation

Voir les instructions étape par étape fournies avec les fichiers.

## Exemples d'Utilisation

Dans Claude Desktop, vous pouvez demander :

- "[Exemple en langage naturel 1]"
- "[Exemple en langage naturel 2]"

[PROVIDEZ DES EXEMPLES POUR CHAQUE OUTIL]

## Architecture

Claude Desktop → MCP Gateway → [SERVICE_NAME] MCP Server → [SERVICE/API]
↓
Docker Desktop Secrets
([SECRET_NAMES])

## Développement

### Tests Locaux

bash
# Set environment variables for testing
export [SECRET_NAME]="test-value"

# Run directly
python [SERVER_NAME]_server.py

# Test MCP protocol
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | python [SERVER_NAME]_server.py


### Ajouter Nouvelles Outils

1. Ajoutez la fonction à `[SERVER_NAME]_server.py`.
2. Décorez avec `@mcp.tool()`.
3. Mettez à jour l'entrée du catalogue avec le nouveau nom d'outil.
4. Reconstruisez l'image Docker.

## Dépannage

### Outils Non Apparents

- Vérifiez que l'image Docker s'est construite avec succès.
- Vérifiez les fichiers de catalogue et de registre.
- Assurez-vous que la configuration de Claude Desktop inclut le catalogue personnalisé.
- Redémarrez Claude Desktop.

### Erreurs d'Authentification

- Vérifiez les secrets avec `docker mcp secret list`.
- Assurez-vous que les noms des secrets correspondent dans le code et le catalogue.

## Considérations de Sécurité

- Tous les secrets sont stockés dans les secrets Docker Desktop.
- Ne jamais coder en dur des identifiants.
- Fonctionner en tant qu'utilisateur non-root.
- Les données sensibles ne sont jamais enregistrées.

## Licence

Licence MIT

## Liste de Vérification Finale pour la Génération du LLM

Avant de présenter votre réponse, vérifiez :

- [ ] Création de tous les 5 fichiers avec le nom approprié.
- [ ] Aucun décorateur `@mcp.prompt()` utilisé.
- [ ] Aucun paramètre de prompt dans `FastMCP()`.
- [ ] Aucun type hint complexe.
- [ ] TOUTES les docstrings des outils sont UNIQUEMENT EN LIGNE.
- [ ] TOUS les paramètres par défaut sont des chaînes vides ("") et non None.
- [ ] Tous les outils retournent des chaînes.
- [ ] Vérification des chaînes vides avec .strip() et non uniquement la véracité.
- [ ] Gestion des erreurs dans chaque outil.
- [ ] Séparation claire entre les fichiers et les instructions utilisateur.
- [ ] Tous les espaces réservés remplacés par des valeurs réelles.
- [ ] Exemples d'utilisation fournis.
- [ ] La sécurité gérée via les secrets Docker.
- [ ] Le catalogue comprend version: 2, name, displayName et l'enveloppe du registre.
- [ ] Les entrées du registre se trouvent sous la clé registry: avec ref: "".
- [ ] Le format de date est ISO 8601 (AAAA-MM-JJTHH:MM:SSZ).
- [ ] Le fichier de configuration de Claude n'a pas de commentaires.
- [ ] Chaque fichier apparaît exactement une fois.
- [ ] Les instructions sont claires et numérotées.

```