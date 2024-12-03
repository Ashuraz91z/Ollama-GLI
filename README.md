# Ollama IA Chat Interface

Une interface de chat moderne qui se connecte à Ollama pour des conversations alimentées par l'IA.

## ⚠️ Prérequis Important

Cette application nécessite :

1. Ollama installé sur votre machine
2. Le modèle llama3.2 spécifiquement (autres modèles non supportés)
3. Le serveur Ollama démarré avec la commande :
   ```bash
   ollama serve
   ```

## Caractéristiques

- 🎨 Interface utilisateur moderne avec thème sombre
- 💬 Chat en temps réel
- ✨ Support du Markdown
- 🔄 Animations de chargement
- ⌨️ Raccourcis clavier (Entrée pour envoyer)
- 📱 Design responsive

## Stack Technique

- Vue 3 avec TypeScript
- Tailwind CSS
- Marked.js pour le rendu Markdown
- Highlight.js pour la coloration syntaxique
- Intégration API Ollama

## Configuration

1. Installez Ollama depuis le site officiel
2. Installez le modèle llama3.2 :
   ```bash
   ollama pull llama3.2
   ```
3. Démarrez le serveur Ollama :
   ```bash
   ollama serve
   ```
4. Le serveur doit être accessible sur `localhost:11434`

## Utilisation

- Assurez-vous que le serveur Ollama est en cours d'exécution
- Tapez votre message dans la zone de texte
- Envoyez avec le bouton ou la touche Entrée
- Les réponses sont formatées automatiquement avec support Markdown
- Les messages d'erreur s'affichent si le serveur n'est pas accessible

## Limitations

- Fonctionne exclusivement avec le modèle llama3.2
- Nécessite une connexion locale au serveur Ollama
- Le serveur doit être démarré avant l'utilisation de l'interface

## Interface

- Zone de chat avec historique des messages
- Distinction visuelle entre messages utilisateur et assistant
- Indicateur de chargement pendant le traitement
- Zone de saisie avec bouton d'envoi
- Support du formatage Markdown dans les réponses
