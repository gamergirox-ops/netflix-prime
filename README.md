# Veltrix Music Bot

Bot Discord professionnel avec :

- **Quiz musical horaire** (heure de Paris) — extraits YouTube + boutons
- **Radio** depuis l’URL que tu configures
- **Sondage musique** (Oui / Non → joue ou refuse)
- **Système de points** + barre de progression + classement
- **File d’attente** mise à jour en direct dans un salon

Tout est synchronisé pour éviter les chevauchements (le quiz a la priorité absolue).

---

## Prérequis

- Node.js **18+** (recommandé 20 ou 22)
- Un bot Discord (token + Client ID)
- **FFmpeg** (système ou via `ffmpeg-static`)
- **yt-dlp** (obligatoire pour les extraits YouTube / quiz / sondages)

### Installer yt-dlp (très important)

**Windows :**
```bash
winget install yt-dlp
```
ou télécharge : https://github.com/yt-dlp/yt-dlp/releases

**Linux / Mac :**
```bash
sudo curl -L https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -o /usr/local/bin/yt-dlp
sudo chmod a+rx /usr/local/bin/yt-dlp
```

Vérifie : `yt-dlp --version`

---

## Installation (Windows / Linux / Mac)

1. Extrais le dossier `veltrix`
2. Ouvre un terminal dans le dossier
3. Installe les dépendances :

```bash
npm install
```

4. Copie le fichier d’exemple :

```bash
cp .env.example .env
```

5. Édite le fichier `.env` avec tes valeurs (token, IDs de salons, URL radio, etc.)

6. Déploie les commandes slash :

```bash
npm run deploy
```

7. Lance le bot :

```bash
npm start
```

---

## Configuration Discord

1. Va sur [Discord Developer Portal](https://discord.com/developers/applications)
2. Crée une application → onglet **Bot** → Reset Token → copie le token
3. Active les **Privileged Gateway Intents** :
   - Server Members Intent (optionnel)
   - Message Content Intent
4. Onglet **OAuth2 → URL Generator** :
   - Scopes : `bot` + `applications.commands`
   - Permissions : Connect, Speak, Send Messages, Embed Links, Use Voice Activity, Manage Channels ()
5. Invite le bot sur ton serveur
6. Récupère les IDs (mode développeur Discord) :
   - Clic droit sur un salon → Copier l’identifiant
   - Pareil pour les rôles

---

## Variables importantes du `.env`

| Variable | Description |
|----------|-------------|
| `DISCORD_TOKEN` | Token du bot |
| `CLIENT_ID` | ID de l’application |
| `GUILD_ID` | ID du serveur (pour déploiement rapide) |
| `RADIO_URL` | URL de ton flux radio (mp3/aac/ogg) |
| `QUEUE_CHANNEL_ID` | Salon où la file d’attente est affichée et mise à jour |
| `LEADERBOARD_CHANNEL_ID` | Salon du classement (actualisé toutes les 10 min) |
| `DEFAULT_VOICE_CHANNEL_ID` | Salon vocal par défaut pour les quiz horaires |
| `POINTS_PER_LEVEL` | Points nécessaires pour monter d’un niveau (défaut 100) |

---

## Commandes

| Commande | Description |
|----------|-------------|
| `/radio start` | Lance la radio dans ton salon vocal |
| `/radio stop` | Arrête la radio |
| `/radio status` | État de la radio |
| `/quiz start` | Lance un quiz immédiatement (staff) |
| `/quiz stop` | Arrête le quiz (staff) |
| `/quiz stats` | Tes stats |
| `/minijeux pfc` | Pierre – Feuille – Ciseaux |
| `/minijeux de` | Lancer de dé |
| `/minijeux pileface` | Pile ou face |
| `/minijeux nombre` | Devine un nombre 1-10 |
| `/minijeux mot` | Mot mélangé (rap/pop) |
| `/points profil` | Ton profil + barre de progression |
| `/points classement` | Classement général |
| `/points user @membre` | Profil d’un autre membre |
| `/sondage titre:...` | Lance un sondage Oui/Non pour une musique |
| `/queue voir` | Affiche la file d’attente |
| `/queue skip` | Passe la musique (staff) |
| `/queue clear` | Vide la file (staff) |

---

## Fonctionnement détaillé

### Quiz horaire
- Toutes les heures pile (00, 01, 02… heure de Paris)
- Le bot rejoint un salon vocal, joue un extrait (~18 s) via YouTube
- 4 boutons de réponse, 45 secondes
- Première bonne réponse gagne
- Points : +15 bonne réponse +5 bonus rapidité +2 participation
- Anti-répétition des questions

### Radio
- URL configurée dans `.env`
- Reconnexion automatique en cas de coupure
- Priorité inférieure au quiz

### Sondage musique
- `/sondage titre:Blinding Lights`
- Boutons Oui / Non pendant 60 s
- Si majorité Oui → ajouté à la file d’attente et joué si possible

### Points & Classement
- Barre de progression visuelle
- Classement mis à jour automatiquement dans le salon dédié
- Historique des transactions en base

### File d’attente
- Message unique édité en temps réel dans `QUEUE_CHANNEL_ID`
- Affiche la musique en cours + la liste

---

## Notes importantes

- Les extraits du quiz passent par **YouTube** (via `play-dl`). Certains titres peuvent être géobloqués ou indisponibles.
- La radio doit être un **flux HTTP/HTTPS direct** (pas de page web).
- Le bot ne contourne aucune protection DRM.
- Tu peux ajouter facilement de nouvelles questions dans `src/config/questions.js`.

---

## Structure du projet

```
veltrix/
├── src/
│   ├── commands/       # Slash commands
│   ├── events/         # ready, interactionCreate
│   ├── services/       # Audio, Quiz, Radio, Queue, Poll
│   ├── database/       # SQLite
│   ├── utils/          # embeds, logger
│   ├── config/         # banque de questions
│   └── index.js
├── data/               # base SQLite + logs
├── music/clips/        # pour futurs extraits locaux
├── .env.example
├── deploy-commands.js
└── README.md
```

---

Bon voyage avec Veltrix 🚀

---

## Limites ressources (hébergement léger)

Optimisé pour :
- **RAM** ≤ 256 Mo (`--max-old-space-size=200`)
- **Disque** ≤ 512 Mo (évite les gros caches)
- **CPU** ≤ 50 % (décompte quiz toutes les 5s, classement toutes les 30 min)

Astuce disque : si FFmpeg est déjà installé sur le serveur, tu peux retirer `ffmpeg-static` de `package.json` pour gagner ~50-80 Mo.
