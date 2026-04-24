<div align="center">

```
  ███████╗███╗   ██╗ ██████╗ ██████╗ ███████╗   ███████╗ ██████╗ ██████╗ ██████╗ ███████╗
  ██╔════╝████╗  ██║██╔═══██╗██╔══██╗██╔════╝   ██╔════╝██╔════╝██╔═══██╗██╔══██╗██╔════╝
  ███████╗██╔██╗ ██║██║   ██║██████╔╝█████╗     ███████╗██║     ██║   ██║██████╔╝█████╗
  ╚════██║██║╚██╗██║██║   ██║██╔══██╗██╔══╝     ╚════██║██║     ██║   ██║██╔═══╝ ██╔══╝
  ███████║██║ ╚████║╚██████╔╝██║  ██║███████╗   ███████║╚██████╗╚██████╔╝██║     ███████╗
  ╚══════╝╚═╝  ╚═══╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝   ╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚══════╝

                       .  *  .        ☽        .  *  .
                    *        .                .        *
                         ~  z Z z  ~  z Z z  ~
                      ─────────────────────────────
```

**Une PWA qui enregistre et analyse tes ronflements. Directement dans Safari.**

[Lancer l'app →](https://prowotion.github.io/snore-app/)

</div>

---

## Pourquoi

Les apps de suivi du sommeil sont partout sur l'App Store. Elles t'envoient ton audio nocturne dans le cloud, te demandent un abonnement, te bombardent de notifications. J'en voulais une :

- **Sans App Store**, sans TestFlight, sans compte.
- **100% locale** — les sons ne quittent jamais l'iPhone.
- **Installable en 10 s** via une URL dans Safari.
- **Pilotable** : le code est là, les seuils sont au début du fichier.

## Fonctionnalités

- 🎙️ **Enregistrement continu** toute la nuit
- 🔍 **Détection acoustique** des ronflements (RMS + FFT 60–400 Hz, durée 0,3–3 s)
- 📊 **Snore Score 0–100** — fréquence × intensité × % de nuit bruyante
- 💨 **Régularité respiratoire** — stabilité estimée via la régularité des cycles
- ⏸️ **Détection de pauses** > 10 s (marqueurs possibles d'apnée — non médical)
- 📈 **Fréquence respiratoire** estimée
- 🗂️ **Historique** des nuits + comparatif
- ▶️ **Replay** de chaque ronflement détecté
- 🌚 **Mode auto-sombre** : écran noir après 12 s, un micro-point indique l'activité
- 🔒 **Stockage local** (IndexedDB), aucun serveur, aucune télémétrie

## Installation sur iPhone

1. Ouvre [prowotion.github.io/snore-app](https://prowotion.github.io/snore-app/) dans **Safari** (pas Chrome).
2. Bouton **Partager** → **Ajouter à l'écran d'accueil**.
3. Lance depuis l'icône "SnoreScope".
4. Au premier démarrage, autorise le micro.

### Pour la nuit

| ✅ À faire                              | ❌ À éviter                          |
| --------------------------------------- | ----------------------------------- |
| Brancher le téléphone au secteur        | Verrouiller l'écran manuellement    |
| Poser à 30–50 cm de la tête             | Mettre sous l'oreiller              |
| Baisser la luminosité iOS au minimum    | Changer d'app pendant la nuit       |
| Appuyer sur **Commencer** avant de dormir | Couper les autorisations micro      |

> **Limite iOS Safari** : si tu verrouilles l'écran ou passes sur une autre app, iOS suspend le JS et l'enregistrement s'arrête. Wake Lock empêche la mise en veille automatique, pas le verrouillage manuel.

## Stack technique

- HTML / CSS / JS vanilla — **zéro dépendance**, un seul fichier
- **Web Audio API** (`AnalyserNode`, FFT 2048)
- **MediaRecorder** (AAC sur iOS, un enregistrement continu par nuit)
- **IndexedDB** pour le stockage (blob + events + métriques)
- **Wake Lock API** pour garder l'écran allumé
- **Service Worker** pour le mode hors-ligne + updates automatiques (network-first)
- **Web App Manifest** pour l'install PWA

## Ajuster la détection

Les paramètres sont en haut de `SnoreSession._tick` dans [`index.html`](index.html) :

```js
const loudEnough = rms > 0.018;         // seuil volume
const snoreLike  = loudEnough && lowRatio > 0.42;  // dominance basse fréquence
// ...
if (dur >= 0.3 && dur <= 3.5) {          // durée valide d'un ronflement
```

Baisse `0.018` si ton micro est peu sensible. Monte `0.42` si tu as trop de faux positifs (TV, voix, etc).

## Clonage / contribution

```bash
git clone https://github.com/prowotion/snore-app.git
cd snore-app
python3 -m http.server 8080
# puis : http://localhost:8080 (Chrome/Safari desktop — mic OK sur localhost)
```

Pour tester depuis un iPhone sans déployer :

```bash
npx cloudflared tunnel --url http://localhost:8080
```

## Avertissement

Ce n'est **pas un dispositif médical**. Les métriques (régularité respiratoire, pauses) sont des estimations acoustiques. Si tu suspectes une apnée du sommeil, consulte un médecin.

## Audit & transparence

Le projet est volontairement **auditable par toute personne ou agent IA** :

- **Public et sous licence MIT** — utilisation, fork, modification, audit libres ([LICENSE](LICENSE)).
- **Zéro dépendance, zéro build** — ce que tu lis dans `index.html` est exactement ce qui tourne sur ton iPhone. Pas de bundler, pas de transpilation, pas de minification.
- **[`llms.txt`](llms.txt)** à la racine — index lisible par les LLMs ([spec](https://llmstxt.org/)) décrivant la structure, les formules de score et les zones-clés à auditer.
- **[`robots.txt`](robots.txt)** — crawlers IA (GPTBot, ClaudeBot, Google-Extended, PerplexityBot, CCBot, etc.) explicitement autorisés.
- **Zéro serveur** — aucun code backend à auditer. L'app entière tient dans 3 fichiers.

Pour auditer rapidement :

```bash
# Récupère les sources brutes
curl -sO https://raw.githubusercontent.com/prowotion/snore-app/main/index.html
curl -sO https://raw.githubusercontent.com/prowotion/snore-app/main/sw.js
curl -sO https://raw.githubusercontent.com/prowotion/snore-app/main/llms.txt
```

Ou pose-le à n'importe quelle IA :

> Audite https://github.com/prowotion/snore-app : vérifie l'absence de fuite réseau des données audio, la cohérence de l'algorithme de détection et les formules de score. Commence par llms.txt.

## License

[MIT](LICENSE) — fais-en ce que tu veux.
