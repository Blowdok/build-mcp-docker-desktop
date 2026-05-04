<p align="center">
  <img src="image/image-readme.png" alt="Blowdok MCP Server Builder" width="100%" />
</p>

# Blowdok MCP Server Builder

Un ensemble de prompts et d'instructions structurés pour générer rapidement des serveurs **MCP (Model Context Protocol)** complets, fonctionnels et compatibles **Docker Desktop** + **Claude Desktop**.

## Objectif

Ce projet fournit un cadre standardisé permettant à un LLM (comme Claude) de produire, à la demande, un serveur MCP prêt à l'emploi en respectant des règles strictes qui évitent les erreurs connues de Claude Desktop (panic gateway, docstrings multilignes, type hints complexes, etc.).

## Contenu du dépôt

| Fichier | Description |
|---------|-------------|
| `INSTRUCT.md_2026-02-03.md` | Spécification complète du builder : règles de génération, structure des 5 fichiers à produire (Dockerfile, requirements.txt, `*_server.py`, README, CLAUDE.md) et instructions d'installation pas-à-pas. |
| `Instruction personnalisé_2026-03-23.md` | Définition du rôle de l'assistant et de la commande `/ServeurMCP` qui déclenche le processus de clarification puis de génération. |

## Comment utiliser

1. Charge les deux fichiers d'instructions dans la base de connaissance de ton assistant (Claude Desktop, projet Claude, etc.).
2. Dans une conversation, lance la commande :
   ```
   /ServeurMCP
   ```
3. L'assistant te posera des questions de clarification :
   - Nom du service
   - Documentation API à intégrer
   - Liste des outils/fonctions souhaités
   - Mode d'authentification (API key, OAuth, etc.)
   - Sources de données
4. Une fois les réponses fournies, l'assistant génère :
   - `Dockerfile`
   - `requirements.txt`
   - `<server_name>_server.py`
   - `readme.txt`
   - `CLAUDE.md`
5. Suis les instructions d'installation (build Docker, secrets, catalogue `~/.docker/mcp/catalogs/custom.yaml`, registre, configuration Claude Desktop).

## Règles critiques imposées au générateur

- Aucun décorateur `@mcp.prompt()`
- Aucun paramètre `prompt` dans `FastMCP()`
- Aucun type hint depuis `typing` (`Optional`, `Union`, `List`, …)
- Docstrings **uniquement sur une seule ligne**
- Valeurs par défaut = chaînes vides (`param: str = ""`), jamais `None`
- Tous les outils retournent une **chaîne formatée**
- Exécution **obligatoire dans Docker**
- Logs sur **stderr**
- Gestion d'erreurs systématique avec messages conviviaux

## Prérequis pour les serveurs générés

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) avec le **MCP Toolkit** activé
- Plugin CLI `docker mcp`
- Claude Desktop (ou un autre client MCP compatible)

## Architecture cible

```
Claude Desktop
      │
      ▼
 MCP Gateway (docker/mcp-gateway)
      │
      ▼
 Serveur MCP généré (conteneur Docker)
      │
      ▼
 API / Service externe
```

Les secrets sont gérés via `docker mcp secret set` et injectés au runtime — jamais codés en dur.

## Remerciements

Une dédicace spéciale à **[NetworkChuck](https://www.youtube.com/@NetworkChuck)** 🙌
Ce projet a été rendu possible grâce à sa vidéo : [Watch on YouTube](https://www.youtube.com/watch?v=GuTcle5edjk).
Merci pour la pédagogie, l'énergie et l'inspiration !

## Licence

MIT
