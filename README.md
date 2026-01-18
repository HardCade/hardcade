## 📥 Téléchargement

Voir les [Releases](../../releases) pour télécharger.

# CRT-MAME-ARCADE-2D Perceptual Sync 0.168 V2.1

**Version Développement — Décembre 2025**  
© 2025 Hardcade — Olivier Mileo

---

## DESCRIPTION

CRT-MAME-ARCADE-2D Perceptual Sync est une version spécialisée de MAME 0.168 optimisée pour l'arcade 2D sur moniteurs CRT 15 kHz sous WINXP-32 avec carte graphique ATI ou NVIDIA + crt_emu drivers ou soft15khz.

### Trois approches complémentaires de synchronisation et sélection vidéo CRT

Trois approches sont proposées pour un contrôle total de votre affichage :

#### Option INI `auto_refresh_sync = 0`

**→ MODE MANUEL PERFECT SYNC** : Affinage précis du Slider Refresh Rate jusqu'à 4 décimales (±0.0001 Hz) pour un contrôle total du timing. Idéal pour les puristes qui souhaitent calibrer chaque jeu individuellement et obtenir un timing hardware strictement authentique avec un tearing fixe quasi-invisible.

#### Option INI `auto_refresh_sync = 1`

**→ MODE AUTOMATIQUE PERFECT SYNC (DDRAW)** : Le refresh rate réel du CRT est mesuré automatiquement au lancement du jeu, après que l'écran vidéo ait produit un nombre suffisant de frames stables. La valeur calculée est appliquée dynamiquement au moteur vidéo et au slider utilisateur en mémoire uniquement (aucune écriture dans le fichier CFG). Scrolling ultra-fluide même sur des modelines atypiques (57.45878 Hz, 58.25494 Hz, 60.61575 Hz…)

#### HARDCADE SWITCHRES CRT (Intégré)

**→ SÉLECTION DE MODE OPTIMISÉE 15KHZ** : Une réécriture profonde de l'algorithme DirectDraw. Contrairement au MAME standard, cette version privilégie la fluidité (Hz) sur la résolution exacte grâce à une tolérance verticale de ±24 lignes. Elle permet d'utiliser vos meilleures modelines 240p pour tout le catalogue (224p, 239p, 256p...) afin de garantir un affichage toujours complet, centré, et une synchronisation verticale parfaite sans saccades.

### Choisissez votre philosophie

Contrôle manuel absolu ou fluidité automatique sans effort. Les deux méthodes éliminent le tearing mobile et garantissent une expérience CRT optimale.

- Si vos modelines sont proches des timings originaux des jeux, l'activation de cette option vous offrira l'expérience ultime.
- Si vos modelines sont éloignées des timings originaux, vous percevrez uniquement une légère augmentation ou diminution de la vitesse du jeu, tout en conservant un timing parfait en termes de scrolling fluide, d'élimination du tearing, et de fidélité visuelle globale.

---

## PHILOSOPHIE

### Pourquoi "CRT-MAME-ARCADE-2D Perceptual Sync" ?

Que vous émulez en 15khz avec du matériel récent ou ancien, la synchronisation entre l'émulation et l'affichage n'est jamais parfaite à 100% en pratique. Même si le modeline est calculé pour correspondre exactement à la fréquence du jeu original, parfois même en respectant les valeurs précises des drivers video de MAME, le timing interprété par votre matériel sera plus ou moins éloigné du timing qu'il devrait réellement adopter pour être parfaitement calé sur le timing du jeu.

Il existe toujours des écarts (grands ou minuscules) dus aux :

- Tolérances du matériel (moniteur CRT, carte graphique)
- Imprécisions dans les horloges et oscillateurs
- Arrondis dans les calculs, système d'exploitation

Ces écarts sont aléatoires selon votre matériel même si vous utilisez LA modeline parfaite qui respecte mathématiquement les valeurs imposées par le système du jeu MAME. Malgré ça, la synchronisation dérive plus ou moins au fil du temps, créant cette ligne de tearing qui "marche" lentement ou rapidement sur l'écran - elle peut mettre plusieurs minutes voire dizaines de minutes pour traverser tout l'écran.

C'est généralement considéré comme acceptable car :

- La ligne se déplace si lentement qu'elle est peu gênante en jeu
- C'est infiniment mieux que du tearing classique avec des lignes multiples qui bougent rapidement
- En y ajoutant une V-sync on la fait disparaître au détriment d'une frame d'input lag, mais si notre modeline est trop éloignée du timing parfait nous obtiendrons un scrolling saccadé

Certains utilisateurs affinent encore leurs modelines ou ajustent légèrement la fréquence de rafraîchissement pour minimiser ce phénomène, mais un micro-tearing reste souvent présent.

### Définition de Perceptual Sync

**Perceptual Sync** est la philosophie d'un mode d'affichage CRT qui privilégie la stabilité visuelle perçue (zéro tearing mobile, scrolling fluide) plutôt que l'exactitude absolue du refresh théorique. C'est une doctrine d'affichage CRT basée sur la perception humaine, pas sur la perfection mathématique.

Perceptual Sync privilégie la fluidité perçue et la stabilité de l'image sur CRT. De légères variations de vitesse (≤0,0001 à 0,5 Hz) sont volontairement acceptées afin d'éliminer le tearing mobile.

### 🔴 Ce que Perceptual Sync NE CHERCHE PAS à faire

- ❌ Être mathématiquement exact
- ❌ Être "perfect frame"
- ❌ Imiter GroovyMAME
- ❌ Convaincre les puristes théoriques

**👉 Il assume ses choix.**

Autrement dit : **"Ce que l'œil voit est plus important que ce que les chiffres disent."**

### Les principes fondamentaux

**Le refresh n'a PAS besoin d'être exact si on tolère une variation de : ±0.0001 à 0,5 Hz (configurable)**

👉 Résultat : vitesse imperceptiblement différente, image stable.

**Priorité absolue à la stabilité du tearing**

- Tearing autorisé ou pas avec Vsync activée
- Mais : fixe, coincé hors zone visible si possible ou toujours au même endroit

👉 Un tearing immobile est psychologiquement invisible.

**Aucune chasse au "modeline parfaite"**

- Pas de calcul dynamique
- Pas de création de modes
- Pas d'ajustement en temps réel

👉 Une fois le mode choisi → et le slider refresh rate affiné on n'y touche plus !

**Le joueur prime sur le chronomètre**

- L'émulation respecte le gameplay
- Pas l'horloge atomique
- Aucune dérive perceptible en jeu

---

## PRINCIPALES FONCTIONNALITÉS

### INPUT & LATENCE

- **Late Input Polling** — Réduction de l'input lag
- **DirectInput non bufferisé** — Polling direct de l'état des périphériques
- **Suppression de la frame queue GPU (D3D9)**

### VIDEO & SYNCHRONISATION

- **Auto Refresh Syncro** — Calcul du réel refresh rate, s'adapte AUTOMATIQUEMENT au refresh réel du CRT
- **Sauvegarde du slider Screen Refresh Rate** dans les CFG + précisions à 4 décimales au lieu de 3 par défaut
- **Switchres** - HARDCADE CRT-OPTIMIZED MODE SELECTION (Version 3.0)
- **DirectDraw Low-Level VSync** — Suppression du tearing sans surcoût
- **D3D9 Real VSync** — Suppression du tearing sans surcoût
- **Désactivation du frameskip implicite** — Scrolling fluide sur CRT

### INTERFACE & CONFIGURATION

- **Build ARCADE 2D optimisé** — Exécutable allégé (dépourvu de jeux 3D, mécanique, casino, mahjong, ordi, consoles... No Open GL, No BGFX, No support Network, No sound midi, No LUA Script)

---

## MODIFICATIONS TECHNIQUES DÉTAILLÉES

### AUTO REFRESH SYNC CRT / LCD DDRAW (HARDCADE)

Cette fonctionnalité permet de mesurer et synchroniser automatiquement le refresh rate réel du CRT au lancement du jeu.

La calibration est effectuée UNE SEULE FOIS, après le démarrage effectif du jeu, lorsque l'écran vidéo est stable (frames réelles déjà produites).

Le refresh mesuré est :
- appliqué dynamiquement au moteur vidéo
- appliqué au slider utilisateur en mémoire uniquement

Lorsque `auto_refresh_sync` est activé, toute option de sauvegarde du slider refresh dans le CFG est volontairement ignorée.

Cette approche garantit :
- synchronisation exacte CRT ↔ jeu
- scrolling parfaitement fluide
- absence de tearing
- comportement déterministe à chaque lancement

La calibration est volontairement retardée de plusieurs dizaines de frames afin d'éviter toute mesure instable lors des phases d'initialisation.

**Option INI :**
- `auto_refresh_sync 1`

**Comportement :**
- Fonction active uniquement lorsque `auto_refresh_sync = 1`
- Le refresh est recalculé à chaque lancement du jeu
- Le slider utilisateur reste modifiable pendant l'exécution
- Compatible lancement direct par ligne de commande
- Indépendant de l'UI MAME

**Fichiers modifiés :**
- `src/emu/video.cpp`
- `src/emu/video.h`
- `src/emu/ui/ui.cpp`
- `src/osd/windows/drawdd.cpp`

---

### AUTO REFRESH SYNC CRT / LCD D3D (HARDCADE)

*... En cours de dev ...*

---

### SLIDER SCREEN REFRESH RATE — HAUTE PRÉCISION & SAUVEGARDE CFG

Sauvegarde et rechargement automatique de la fréquence utilisateur dans `cfg/[nom_du_jeu].cfg`, avec gestion du réglage ultra-fin du refresh rate CRT à 0.0001 Hz, affichage et sauvegarde à 4 décimales.

**Fonctionne uniquement si `auto_refresh_sync = 0`**

**Fonctionnement des touches :**
- Flèches seules → ±1.0000 Hz
- SHIFT + flèches → ±0.1000 Hz
- ALT + flèches → ±0.0010 Hz
- ESPACE + flèches → ±0.0001 Hz
- CTRL + flèches → ±1.0000 Hz (rapide)

**Fonctions ajoutées :**
- `config_load_screen_refresh()` — `src/emu/video.cpp`
- `config_save_screen_refresh()` — `src/emu/video.cpp`

**Fonctions modifiées :**
- `slider_refresh()` — `src/emu/ui/ui.cpp`
  - Conversion base 10000 → Hz pour 4 décimales
  - Arrondi et sauvegarde CFG précis à 4 décimales
  - Affichage FPS en 4 décimales
- `ui_menu_sliders::handle()` — `src/emu/ui/sliders.cpp`
  - Gestion de la touche ESPACE pour incrément ultra-fin
- `slider_init()` — `src/emu/ui/ui.cpp`
  - incval du slider refresh modifié à 1 (0.0001 Hz)

**Fichiers concernés :**
- `src/emu/screen.cpp` / `screen.h`
- `src/emu/video.cpp`
- `src/emu/ui/ui.cpp`
- `src/emu/ui/sliders.cpp`

---

### HARDCADE SWITCHRES - CRT-OPTIMIZED MODE SELECTION

Cette version de CRT-MAME-ARCADE 0.168 introduit une réécriture complète de la fonction SwitchRes DirectDraw, spécifiquement pensée pour un affichage CRT 15 kHz (arcade / JAMMA / RGB) et les configurations Windows XP / GeForce.

La sélection du mode vidéo DirectDraw repose désormais sur une hiérarchie de priorités "Hardware-First", garantissant la fluidité et l'intégrité de l'image.

#### LOGIQUE DE SÉLECTION HARDCADE V3

**1. PRIORITÉ ABSOLUE AU "INI" (User Override)**

Si une résolution est forcée dans le fichier .ini du jeu, le SwitchRes l'applique immédiatement avec un score prioritaire (1.000.000 pts). Cela permet de forcer un mode spécifique même s'il est techniquement éloigné de la résolution native du jeu.

**2. TOLÉRANCE VERTICALE INTELLIGENTE (±24 Lignes)**

En mode Auto, le SwitchRes autorise désormais un écart allant jusqu'à 24 lignes verticales (ex: utiliser un mode 240p pour un jeu en 224p). Cela permet de maintenir l'utilisation de modelines stables et bien cadrées physiquement, évitant les écrans noirs ou les images tronquées.

**3. PRIORITE DU REFRESH RATE (Fluidité Totale)**

À l'intérieur de la zone de tolérance verticale, c'est la proximité du rafraîchissement (Hz) qui détermine le gagnant. Le système préférera toujours un mode à 58Hz pour un jeu de 58Hz, même si le nombre de lignes diffère légèrement, garantissant un scrolling parfait sans saccades.

#### DIFFÉRENCES AVEC LA LOGIQUE MAME D'ORIGINE

**• MAME ORIGINAL (Logique LCD) :**
- Priorité au refresh supérieur (pénalité si Hz inférieur).
- Rejet immédiat si la résolution est inférieure à la cible (Image tronquée).
- Importance démesurée de la largeur (Pixel Clock).

**• HARDCADE CRT (Logique Analogique) :**
- Hauteur (Scanlines) et Refresh sont les seuls critères vitaux.
- Largeur traitée comme critère secondaire (ajustable sur le moniteur).
- Élimination du scaling vertical destructeur.

**FICHIERS MODIFIÉS :**

- `src/osd/modules/render/drawdd.cpp`
  - → Réécriture de `enum_modes_callback` (Logique Hardcade V3)

**• Rendu LCD** (Fonctionne aussi parfaitement !)

Aucun impact sur :
- D3D / OGL
- Autres backends vidéo

---

### LATE INPUT POLLING

Les entrées DirectInput sont polées le plus tard possible dans la frame, juste avant le rendu vidéo. Évite l'utilisation d'inputs de la frame N-1.

**Fichier modifié :** `src/osd/windows/video.cpp`  
**Fonction :** `windows_osd_interface::update(bool skip_redraw)`

---

### DIRECTINPUT NON BUFFERISÉ

Désactivation du buffer d'événements DirectInput (`DIPROP_BUFFERSIZE = 0`). Lecture directe via `GetDeviceState` élimine 1 à 3 ms de latence.

**Fichier modifié :** `src/osd/windows/input.cpp`

---

### DÉSACTIVATION DU FRAMESKIP IMPLICITE (CRT)

MAME 0.168 applique un frameskip interne même avec `frameskip=0` lorsqu'aucune modification vidéo n'est détectée. Ce comportement dégrade le scrolling CRT.

Le frameskip implicite est désactivé quand `frameskip=0` est explicite. Compatible DDraw, Windows XP, CRT 15 kHz.

**Fichier modifié :** `src/emu/video.cpp`

---

### SUPPRESSION DE LA FRAME QUEUE GPU (D3D9)

Configuration D3D9 :
- `SwapEffect = D3DSWAPEFFECT_COPY`
- `BackBufferCount = 1`

**Résultat :** -1 frame de latence réelle côté affichage.

**Fichier modifié :** `src/osd/windows/ddrawd3d.cpp`

---

### D3D9 REAL VSYNC (NO TEARING)

Force un VSync matériel D3D9 réel (`PresentationInterval = D3DPRESENT_INTERVAL_ONE`) indépendant du VSync MAME classique. Supprime le tearing sans surcoût CPU.

**Option INI :** `crtvsync 0|1`
- `0` — Comportement MAME d'origine (défaut)
- `1` — VSync D3D9 matériel activé

**Usage recommandé :** Écrans LCD 31kHz ou CRT (ajoute ~1 frame de lag sur CRT)

**Fichiers modifiés :**
- `src/osd/windows/winmain.cpp`
- `src/osd/windows/video.h` / `video.cpp`
- `src/osd/windows/ddrawd3d.cpp`

---

### DIRECTDRAW LOW-LEVEL VSYNC (CRT) V3

Mode VSync DirectDraw bas niveau pour CRT 15 kHz sous Windows XP. Rendu direct dans la surface primaire synchronisé sur le Vertical Blank.

**Option INI :** `ddraw_lowlevel_vsync 0|1`
- `0` — Comportement MAME classique (défaut)
- `1` — VSync DirectDraw bas niveau

**⚠ Active uniquement si `waitvsync = 0`**  
**⚠ Désactivé automatiquement si triple buffering actif**

**Fonctionnement :**

1. Attente du VBL via `WaitForVerticalBlank(DDWAITVB_BLOCKEND)`
2. Lock direct de la surface primaire
3. Scan des primitives (détection blending/alpha)
4. Rendu membuffer ou direct selon besoins
5. Copie contrôlée membuffer → surface primaire
6. Unlock + bypass du blit MAME
7. Désactivation complète du throttling MAME (update_throttle bypass)

**Synchronisation automatique type Snes9x :**

- MAME s'adapte AUTOMATIQUEMENT au refresh réel du CRT
- Fonctionne avec N'IMPORTE QUELLE modeline (58 Hz, 60 Hz, 61 Hz...)
- Le VBlank CRT dicte le timing → scrolling toujours fluide
- Pas besoin d'ajuster le slider refresh rate manuellement
- La vitesse d'émulation s'ajuste pour matcher le refresh de l'écran

**Exemple :**
```
Jeu CPS2 natif : 59.637 Hz
Modeline CRT   : 60.000 Hz
→ MAME tourne à 60 Hz (100.6% vitesse) → scrolling parfaitement fluide

Jeu CPS2 natif : 59.637 Hz  
Modeline CRT   : 58.000 Hz
→ MAME tourne à 58 Hz (97.3% vitesse) → scrolling parfaitement fluide
```

**Support :**
- Formats 8-bit, 16-bit, 32-bit
- Blending et effets alpha complets
- RGB 32-bit (0x00ff0000), 16-bit 565 (0xf800), 15-bit 555 (0x7c00)
- Gestion surfaces perdues (Alt+Tab)

**Compatibilité GPU (Windows XP) :**
- ✓ **Excellente :** NVIDIA TNT/GeForce 2/3/4/FX, ATI Radeon 7000-9800, Matrox G200/G400/G450/G550
- ~ **Partielle :** Intel iGPU i815/i845/i865 (VBL émulé)
- ✗ **Non testé :** Drivers Vista+ / WDDM

**Résultat :**
- Latence réduite d'environ 1 frame
- Synchronisation CRT parfaite automatique (type Snes9x/RetroArch)
- Zéro tearing, scrolling ultra-fluide quelle que soit la modeline
- Plus besoin d'affiner manuellement chaque jeu

**Fichiers modifiés :**
- `src/osd/modules/render/drawdd.cpp`
- `src/emu/video.cpp` (update_throttle bypass)
- `src/emu/emuopts.cpp` / `emuopts.h`

---

### ARRONDI AUTOMATIQUE DU REFRESH (ONE-SHOT) — (FONCTION SUPPRIMÉE)

*... (FONCTION SUPPRIMÉE) ...*

---

### BUILD 2D OPTIMISÉE — NETTOYAGE ARCADE.LST

Version allégée spécialisée pour l'arcade 2D classique sur CRT 15 kHz.

**Systèmes supprimés :**
- ✗ Tous les jeux 3D (Model 2/3, Naomi, Taito Type X etc.)
- ✗ Systèmes casino (machines à sous, poker vidéo)
- ✗ Jeux de mahjong
- ✗ Systèmes "machine" non-arcade (ordinateurs, consoles)
- ✗ Systèmes jeux mécaniques etc.

**Systèmes conservés (2D uniquement) :**
- ✓ Capcom CPS1/CPS2/CPS3
- ✓ Neo Geo MVS
- ✓ Konami (GX, Classic)
- ✓ Sega System 16/18/24
- ✓ Taito (F2, F3)
- ✓ Cave (CV1000, PGM)
- ✓ Irem M72/M92
- ✓ Toaplan, Psikyo, Data East
- ✓ Namco System 1/2
- ✓ Classiques 8-bit (Pac-Man, Donkey Kong, Galaga, etc.)

**Résultat :** Exécutable réduit, compilation plus rapide, liste ciblée CRT 2D.

**Fichier modifié :** `src/mame/arcade.lst`

---

### DÉSACTIVATION DE UI LUA

L'appel périodique Lua `periodic_check` et `frame_hook` est maintenant commenté pour éviter tout impact sur les performances ou les menus.

---

## COMPATIBILITÉ

- **OS :** Windows XP / 7 / 8 / 10
- **Renderers :** DDraw (XP), D3D9, GDI
- **Monitors :** CRT 15 kHz, LCD 31 kHz

---

## CRÉDITS

- **Développement :** Olivier Mileo
- **Projet :** CRT-MAME ARCADE-2D 0.168 Edition
- **Base :** MAME 0.168

© 2025 Hardcade — Tous droits réservés
