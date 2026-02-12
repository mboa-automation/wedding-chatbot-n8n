# 🤵💍 Wedding Chatbot — Agent IA avec RAG, n8n & OpenAI

> Chatbot intelligent de gestion des invités pour un mariage, intégrant un agent IA autonome, une architecture RAG (Retrieval-Augmented Generation), une mémoire conversationnelle et un moteur de recherche vectorielle — déployé sur WhatsApp via n8n.

---

## 📌 Présentation du projet

Ce projet est un **agent IA conversationnel complet** connecté à WhatsApp. Il va bien au-delà d'un simple chatbot à règles fixes : il intègre **OpenAI GPT**, une **base vectorielle Pinecone**, et une **mémoire de conversation** pour comprendre chaque invité, gérer son parcours personnalisé et répondre intelligemment à toutes ses questions sur l'événement.

**Cas d'usage concret :** Un invité envoie un message WhatsApp → le bot l'identifie (nouveau ou connu) → il l'inscrit ou reprend sa conversation → si l'invité pose une question, l'agent IA consulte la base de connaissances vectorielle et génère une réponse contextuelle précise — tout cela automatiquement, sans intervention humaine.

---

## 🏗️ Architecture du Workflow

![Workflow n8n — Wedding Chatbot](./workflow_bot-mariage.PNG)

### Description complète des nœuds

```
[WhatsApp Webhook]
        │  Réception du message entrant (POST)
        ▼
   [Input Table]
        │  Mise en forme et structuration des données brutes
        ▼
    [Filtre]
        │  Validation et filtrage des messages non pertinents
        ▼
[Extract Message Data]
        │  Parsing : numéro expéditeur, contenu du message, timestamp
        ▼
[Check User in Sheet]  ◄── Google Sheets : l'invité existe-t-il déjà ?
        │
        ├──── NOUVEL INVITÉ ──────────────────────────────────┐
        │                                                     ▼
        │                                        [Create New User]
        │                                     Ajout d'une ligne dans Google Sheets
        │                                                     │
        │                                                     ▼
        │                                        [Set Question 1]
        │                                     Envoi de la 1ère question
        │                                                     │
        │                                                     ▼
        │                                        [HTTP Request2]
        │                                     POST → Envoi via WhatsApp API
        │
        ├──── INVITÉ CONNU ───────────────────────────────────┐
        │                                                     ▼
        │                                   [Question Flow Logics]
        │                                   Détermine l'étape du parcours
        │                                   (Q1 répondue ? Q2 ? FAQ ?)
        │                                                     │
        │                                    ┌────────────────┴───────────────┐
        │                                    ▼                                ▼
        │                         [Update User Status]               [FAQ Mode?]
        │                         Mise à jour Google Sheets                   │
        │                                                                      ▼
        │                                                             [AI Agent1] ◄─────────────┐
        │                                                        Agent IA autonome              │
        │                                                  (orchestration des outils)           │
        │                                                             │                         │
        │                                           ┌────────────────┼────────────────┐         │
        │                                           ▼                ▼                ▼         │
        │                                    [OpenAI Chat      [Simple          [Pinecone       │
        │                                       Model]         Memory1]        Vector Store]    │
        │                                    GPT pour la    Mémoire de la    Base vectorielle   │
        │                                    génération     conversation      de la FAQ         │
        │                                    de réponses    en cours              │             │
        │                                                                         ▼             │
        │                                                                  [Embeddings          │
        │                                                                    OpenAI]            │
        │                                                                  Vectorisation        │
        │                                                                  des requêtes         │
        │                                                                         │             │
        │                                                                  [Recursive           │
        │                                                                  Character Text       │
        │                                                                  Splitter]            │
        │                                                                  Découpage des        │
        │                                                                  documents FAQ        │
        │
        └──── FILTRE SPÉCIAL ─────────────────────────────────┐
                                                              ▼
                                                    [HTTP Request7]
                                                    POST → cas particuliers
                                                    (messages hors flux normal)
```

---

## ✨ Fonctionnalités

| Fonctionnalité | Description |
|---|---|
| 📱 Intégration WhatsApp | Réception et envoi de messages via Webhook WhatsApp Business |
| 👤 Gestion de sessions | Détecte si l'invité est nouveau ou déjà enregistré dans Google Sheets |
| 💬 Flux de conversation multi-étapes | Guide l'invité à travers un parcours de questions personnalisées |
| 🧠 Agent IA autonome | AI Agent n8n orchestrant les outils GPT, mémoire et Pinecone |
| 🤖 OpenAI GPT intégré | Génération de réponses naturelles et contextuelles via ChatGPT |
| 💾 Mémoire conversationnelle | L'agent se souvient du contexte de la conversation en cours |
| 🔍 RAG — Recherche vectorielle | Recherche sémantique dans la FAQ via Pinecone Vector Store |
| 📐 Embeddings OpenAI | Vectorisation des questions pour une recherche par sens et non par mots-clés |
| 📄 Text Splitter | Découpage intelligent des documents FAQ pour l'indexation vectorielle |
| 📊 Google Sheets | Base de données des invités avec mise à jour du statut en temps réel |

---

## 🛠️ Technologies utilisées

| Technologie | Rôle |
|---|---|
| **[n8n](https://n8n.io/)** | Moteur d'automatisation no-code/low-code — orchestration du workflow |
| **WhatsApp Business API** | Canal de communication principal avec les invités |
| **OpenAI GPT** | Modèle de langage pour la compréhension et la génération de réponses |
| **Pinecone Vector Store** | Base de données vectorielle pour la recherche sémantique dans la FAQ |
| **OpenAI Embeddings** | Transformation des textes en vecteurs pour la recherche par sens |
| **Recursive Text Splitter** | Découpage intelligent des documents pour l'indexation |
| **Simple Memory (n8n)** | Mémoire conversationnelle pour maintenir le contexte |
| **Google Sheets API** | Stockage et mise à jour des fiches invités |
| **Webhooks HTTP** | Déclencheurs et émetteurs de requêtes pour les appels API |

---

## 🧠 Qu'est-ce que le RAG ?

Ce bot utilise une architecture **RAG (Retrieval-Augmented Generation)** :

1. **La FAQ de l'événement** est découpée en morceaux (Text Splitter) et transformée en vecteurs numériques (Embeddings OpenAI)
2. Ces vecteurs sont stockés dans **Pinecone** (base de données vectorielle)
3. Quand un invité pose une question, sa question est aussi transformée en vecteur
4. Pinecone cherche les morceaux de FAQ **les plus proches par sens** (pas juste par mots-clés)
5. Ces morceaux sont envoyés à **GPT** qui génère une réponse naturelle et précise

> **Résultat :** Le bot comprend "c'est où la cérémonie ?" et "adresse du mariage ?" comme la même question — même si les mots sont différents.

---

## 🚀 Comment utiliser ce projet

### Prérequis
- Un compte [n8n](https://n8n.io/) (cloud ou self-hosted)
- Une clé API [OpenAI](https://platform.openai.com/)
- Un compte [Pinecone](https://www.pinecone.io/) (plan gratuit suffisant)
- Un compte Google avec accès à Google Sheets
- Un accès à l'API WhatsApp Business (Meta for Developers)

### Étapes d'installation

**1. Importer le workflow n8n**
```
- Ouvrir n8n
- Aller dans "Workflows" → "Import from file"
- Importer le fichier workflow.json présent dans ce repo
```

**2. Configurer les credentials dans n8n**
```
- OpenAI : ajouter ta clé API OpenAI
- Pinecone : ajouter ta clé API Pinecone + nom de l'index
- Google Sheets : configurer OAuth2
- WhatsApp : ajouter le token d'accès Meta
```

**3. Préparer la base vectorielle Pinecone**
```
- Créer un index Pinecone (dimension : 1536 pour text-embedding-ada-002)
- Préparer ton fichier FAQ (questions/réponses sur l'événement)
- Lancer une première exécution pour indexer les documents
```

**4. Configurer Google Sheets**
```
- Créer une feuille avec les colonnes :
  Téléphone | Nom | Statut | Réponse Q1 | Réponse Q2 | Date
- Mettre à jour l'ID de la feuille dans les nœuds Google Sheets
```

**5. Connecter WhatsApp Webhook**
```
- Sur Meta for Developers, configurer le webhook entrant
- Coller l'URL du webhook n8n généré
- Tester avec un message WhatsApp
```

---

## 📁 Structure du repo

```
wedding-chatbot-n8n/
│
├── workflow.json                # Export du workflow n8n (importable directement)
├── workflow_bot-mariage.PNG     # Capture d'écran du workflow complet
├── README.md                    # Documentation du projet
└── docs/
    └── setup-guide.md           # Guide de configuration détaillé
```

---

## 👥 Auteurs

Projet développé par **[mboa-automation](https://github.com/mboa-automation)** :

- **Messoa Yene Stephane Erwan**
- **Bryan Vincent Mballa** 

---

## 📄 Licence

Ce projet est open source sous licence MIT — libre d'utilisation, modification et distribution.

---

## 🔗 Liens utiles

- [Documentation officielle n8n](https://docs.n8n.io/)
- [OpenAI API](https://platform.openai.com/docs/)
- [Pinecone — Vector Database](https://docs.pinecone.io/)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [WhatsApp Business API](https://developers.facebook.com/docs/whatsapp)
- [n8n AI Agent Documentation](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/)

---

*Projet réalisé dans le cadre d'un apprentissage pratique de l'automatisation et des agents IA — mboa-automation © 2025*
