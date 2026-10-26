# RAG avec n8n, Supabase (pgvector) et Gemini

Chatbot qui répond **uniquement à partir d'un document PDF**. Projet réalisé dans le cadre de l'Atelier 2 (RAG) à l'ESSAT Gabès.

## Architecture

**Indexation (une seule fois)**

```
PDF (Google Drive) -> découpage en chunks (1000 / 150) -> embeddings (gemini-embedding-001) -> table documents (Supabase pgvector)
```

**Chat**

```
Question -> agent n8n -> outil de recherche (embedding de la question + 4 passages les plus proches) -> réponse Gemini
```

L'agent dispose d'une mémoire de conversation et d'un message système qui lui impose de répondre « Je n'ai pas trouvé cette information dans les documents » quand la réponse n'est pas dans le PDF.

## Captures d'écran

**Workflow d'indexation** : le PDF est téléchargé depuis Drive, découpé, transformé en vecteurs puis stocké dans Supabase.

![Workflow d'indexation](images/indexation-rag.png)

**Workflow de chat** : l'agent interroge la base vectorielle et répond à partir des passages retrouvés (4 passages par question).

![Workflow de chat](images/chat-rag.png)

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `Indexation RAG.json` | Workflow n8n : Drive -> chunks -> embeddings -> Supabase |
| `Chat RAG.json` | Workflow n8n : chat, agent, mémoire, outil de recherche |
| `schema.sql` | Table `documents` et fonction `match_documents` (vecteurs de 3072 dimensions) |

## Prérequis

- Docker et n8n (volume `n8n_data` pour conserver workflows et credentials)
- Un compte Supabase (plan gratuit suffisant)
- Une clé API Gemini (Google AI Studio)
- Un projet Google Cloud avec Google Drive API activée et un client OAuth (redirection `http://localhost:5678/rest/oauth2-credential/callback`)

## Reproduire le projet

1. Dans Supabase, exécuter `schema.sql` dans le SQL Editor.
2. Dans n8n, créer trois credentials : **Supabase** (URL du projet et clé secrète), **Google Drive OAuth2** (client OAuth) et **Google Gemini (PaLM) API** (clé API).
3. Importer `Indexation RAG.json` et `Chat RAG.json` (coller le contenu dans un workflow vide avec Ctrl + V), puis rattacher les credentials dans chaque nœud.
4. Dans `Indexation RAG`, choisir votre PDF dans le nœud **Download file**, puis exécuter le workflow **une seule fois**.
5. Ouvrir `Chat RAG` et poser des questions dans le panneau de chat.

## Points importants

- Le même modèle d'embedding (`gemini-embedding-001`) doit servir à l'indexation et à la recherche, avec la même dimension (3072).
- Relancer l'indexation ajoute les chunks une seconde fois (doublons) : vider la table avant de réindexer.
- Un projet Supabase gratuit est mis en pause après 7 jours d'inactivité : utiliser **Resume project** dans le dashboard.
- Une application Google en mode Test voit son autorisation expirer après 7 jours : refaire **Sign in with Google** dans le credential Drive.
- Les modèles Gemini « preview » peuvent renvoyer des erreurs 503 en cas de forte demande : préférer un modèle stable.

## Ce que ce projet m'a appris

- Concevoir un pipeline RAG complet : découpage, embeddings, base vectorielle, recherche et génération.
- Garder la cohérence entre l'indexation et la recherche (même modèle d'embedding, même dimension de vecteur).
- Contrôler le comportement d'un agent avec un message système, par exemple pour qu'il refuse de répondre hors du document.
- Diagnostiquer des erreurs d'exploitation : projet gratuit mis en pause, modèle surchargé (503), autorisation OAuth expirée.
- Protéger les secrets : clés API, secret OAuth et exports de workflows vérifiés avant publication.

## Sécurité

Aucune clé n'est stockée dans ce dépôt. Ne jamais publier la clé secrète Supabase, la clé Gemini ni le secret du client OAuth Google.
