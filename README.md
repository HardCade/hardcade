## 📥 Téléchargement

Voir les [Releases](../../releases) pour télécharger.
# CRT-MAME-ARCADE-2D Perceptual Sync 0.168 V1.8

**Version Développement — Décembre 2025**  
© 2025 Hardcade — Olivier Mileo

---

## DESCRIPTION

CRT-MAME-ARCADE-2D Perceptual Sync est une version spécialisée de MAME 0.168 optimisée pour l'arcade 2D sur moniteurs CRT 15 kHz et écrans LCD modernes sous WINXP-32 avec carte graphique ATI ou NVIDIA + crt_emu drivers ou soft15khz.

Cette édition se concentre sur la réduction maximale de l'input lag, la légèreté et l'amélioration de la synchronisation vidéo pour une expérience arcade authentique grâce à un affinage précis du Slider Refresh Rate jusqu'à 4 décimales.

---

## PHILOSOPHIE

### Pourquoi "CRT-MAME-ARCADE-2D Perceptual Sync" ?

Que vous fassiez de l'émulation 15khz avec du matériel récent ou ancien, la synchronisation entre l'émulation et l'affichage n'est jamais parfaite à 100% en pratique. Même si le modeline est calculé pour correspondre exactement à la fréquence du jeu original, parfois même en respectant les valeurs précises des drivers video de MAME le timing interprété par votre matériel sera plus ou moins éloigné du timing qu'il devrait réellement adopter pour être parfaitement calé sur le timing du jeu.

Il existe toujours des écarts (grands ou minuscules) dus aux :

- Tolérances du matériel (moniteur CRT, carte graphique)
- Imprécisions dans les horloges et oscillateurs
- Arrondis dans les calculs, système d'exploitation

Ces écarts sont aléatoires selon votre matériel même si vous utilisez LA modeline parfaite qui respecte mathématiquement les valeurs imposées par le système du jeu MAME. Malgré ça, la synchronisation dérive plus ou moins au fil du temps, créant cette ligne de tearing qui "marche" lentement ou rapidement sur l'écran - elle peut mettre plusieurs minutes voire dizaines de minutes pour traverser tout l'écran.

C'est généralement considéré comme acceptable car :

- La ligne se déplace si lentement qu'elle est peu gênante en jeu
- C'est infiniment mieux que du tearing classique avec des lignes multiples qui bougent rapidement
- En y ajoutant une V-sync on la fait disparaître au détriment d'une frame d'input lag, mais si notre modeline est trop éloigné du timing parfait nous obtiendrons un scrolling saccadé
- Certains utilisateurs affinent encore leurs modelines ou ajustent légèrement la fréquence de rafraîchissement pour minimiser ce phénomène, mais un micro-tearing reste souvent présent

### Qu'est-ce que Perceptual Sync ?

**Perceptual Sync** est la philosophie d'un mode d'affichage CRT qui privilégie la stabilité visuelle perçue (zéro tearing mobile, scrolling fluide) plutôt que l'exactitude absolue du refresh théorique.

C'est une doctrine d'affichage CRT basée sur la perception humaine, pas sur la perfection mathématique.

Perceptual Sync privilégie la fluidité perçue et la stabilité de l'image sur CRT. De légères variations de vitesse (≤0,0001 à 0,5 Hz) sont volontairement acceptées afin d'éliminer le tearing mobile.

### 🔴 Ce que Perceptual Sync NE CHERCHE PAS à faire

- ❌ Être mathématiquement exact
- ❌ Être "perfect frame"
- ❌ Imiter GroovyMAME
- ❌ Convaincre les puristes théoriques

**👉 Il assume ses choix.**

> *"Ce que l'œil voit est plus important que ce que les chiffres disent."*

### Les principes fondamentaux

**Le refresh n'a PAS besoin d'être exact**
- Si on tolère une variation de : ±0.0001 à 0,5 Hz (configurable)
- 👉 Résultat : vitesse imperceptiblement différente, image stable

**Priorité absolue à la stabilité du tearing**
- Tearing autorisé ou pas avec Vsync activée
- Mais : fixe, coincé hors zone visible si possible ou toujours au même endroit
- 👉 Un tearing immobile est psychologiquement invisible

**Aucune chasse au "modeline parfaite"**
- Pas de calcul dynamique
- Pas de création de modes
- Pas d'ajustement en temps réel
- 👉 Une fois le mode choisi → et le slider refresh rate affiné on n'y touche plus !

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

- **DirectDraw Low-Level VSync** — Latence minimale sur CRT (Windows XP)
- **D3D9 Real VSync** — Suppression du tearing sans surcoût
- **Désactivation du frameskip implicite** — Scrolling fluide sur CRT
- **Arrondi automatique du refresh** — Optimisation des résolutions
- **Switchres - CRT-OPTIMIZED MODE SELECTION**

### INTERFACE & CONFIGURATION

- **Sauvegarde du slider Screen Refresh Rate** dans les CFG + précisions à 4 décimales au lieu de 3 par défaut
- **Build ARCADE 2D optimisé** — Exécutable allégé (dépourvu de jeux 3D, mécanique, casino, mahjong, ordi, consoles... No Open GL, No BGFX, No support Network, No sound midi, No LUA Script)

---

## MODIFICATIONS TECHNIQUES DÉTAILLÉES

### LATE INPUT POLLING

Les entrées DirectInput sont polées le plus tard possible dans la frame, juste avant le rendu vidéo. Évite l'utilisation d'inputs de la frame N-1.

**Fichier modifié :** `src/osd/windows/video.cpp`  
**Fonction :** `windows_osd_interface::update(bool skip_redraw)`

---

### DIRECTINPUT NON BUFFERISÉ

Désactivation du buffer d'événements DirectInput (DIPROP_BUFFERSIZE = 0). Lecture directe via GetDeviceState élimine 1 à 3 ms de latence.

**Fichier modifié :** `src/osd/windows/input.cpp`

---

### DÉSACTIVATION DU FRAMESKIP IMPLICITE (CRT)

MAME 0.168 applique un frameskip interne même avec frameskip=0 lorsqu'aucune modification vidéo n'est détectée. Ce comportement dégrade le scrolling CRT.

Le frameskip implicite est désactivé quand frameskip=0 est explicite. Compatible DDraw, Windows XP, CRT 15 kHz.

**Fichier modifié :** `src/emu/video.cpp`

---

### SUPPRESSION DE LA FRAME QUEUE GPU (D3D9)

Configuration D3D9 :
- SwapEffect = D3DSWAPEFFECT_COPY
- BackBufferCount = 1

**Résultat :** -1 frame de latence réelle côté affichage.

**Fichier modifié :** `src/osd/windows/ddrawd3d.cpp`

---

### SLIDER SCREEN REFRESH RATE — HAUTE PRÉCISION & SAUVEGARDE

Sauvegarde et rechargement automatique de la fréquence utilisateur dans `cfg/[nom_du_jeu].cfg`, avec gestion du réglage ultra-fin du refresh rate CRT à 0.0001 Hz, affichage et sauvegarde à 4 décimales.

#### Fonctionnement des touches :

| Touche | Incrément |
|--------|-----------|
| Flèches seules | ±1.0000 Hz |
| SHIFT + flèches | ±0.1000 Hz |
| ALT + flèches | ±0.0010 Hz |
| ESPACE + flèches | ±0.0001 Hz |
| CTRL + flèches | ±1.0000 Hz (rapide) |

#### Fonctions ajoutées :

- `config_load_screen_refresh()` — `src/emu/video.cpp`
- `config_save_screen_refresh()` — `src/emu/video.cpp`

#### Fonctions modifiées :

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

### D3D9 REAL VSYNC (NO TEARING)

Force un VSync matériel D3D9 réel (PresentationInterval = D3DPRESENT_INTERVAL_ONE) indépendant du VSync MAME classique. Supprime le tearing sans surcoût CPU.

**Option INI :** `crtvsync 0|1`
- `0` — Comportement MAME d'origine (défaut)
- `1` — VSync D3D9 matériel activé

**Usage recommandé :** Écrans LCD 31kHz ou CRT (ajoute ~1 frame de lag sur CRT)

**Fichiers modifiés :**
- `src/osd/windows/winmain.cpp`
- `src/osd/windows/video.h` / `video.cpp`
- `src/osd/windows/ddrawd3d.cpp`

---

### DIRECTDRAW LOW-LEVEL VSYNC (CRT) V2

Mode VSync DirectDraw bas niveau pour CRT 15 kHz sous Windows XP. Rendu direct dans la surface primaire synchronisé sur le Vertical Blank.

**Option INI :** `ddraw_lowlevel_vsync 0|1`
- `0` — Comportement MAME classique (défaut)
- `1` — VSync DirectDraw bas niveau

⚠️ **Active uniquement si waitvsync = 0**  
⚠️ **Désactivé automatiquement si triple buffering actif**

#### Fonctionnement :

1. Attente du VBL via WaitForVerticalBlank(DDWAITVB_BLOCKEND)
2. Lock direct de la surface primaire
3. Scan des primitives (détection blending/alpha)
4. Rendu membuffer ou direct selon besoins
5. Copie contrôlée membuffer → surface primaire
6. Unlock + bypass du blit MAME

#### Support :

- Formats 8-bit, 16-bit, 32-bit
- Blending et effets alpha complets
- RGB 32-bit (0x00ff0000), 16-bit 565 (0xf800), 15-bit 555 (0x7c00)
- Gestion surfaces perdues (Alt+Tab)

#### Compatibilité GPU (Windows XP) :

| Statut | GPU |
|--------|-----|
| ✓ Excellente | NVIDIA TNT/GeForce 2/3/4/FX, ATI Radeon 7000-9800, Matrox G200/G400/G450/G550 |
| ~ Partielle | Intel iGPU i815/i845/i865 (VBL émulé) |
| ✗ Non testé | Drivers Vista+ / WDDM |

**Résultat :** Latence réduite d'environ 1 frame, synchronisation CRT stable.

**Fichiers modifiés :**
- `src/osd/modules/render/drawdd.cpp`
- `src/emu/emuopts.cpp` / `emuopts.h`

---

### ARRONDI AUTOMATIQUE DU REFRESH (ONE-SHOT)

Au premier lancement, si aucun CFG/slider n'existe, le refresh est arrondi à l'entier le plus proche (ex: 57.445 → 57 Hz, 59.636 → 60 Hz).

**Option INI :** `autorefreshround 1`

Le slider utilisateur reste prioritaire.

**Fichiers modifiés :**
- `src/emu/machine.cpp`
- `src/emu/screen.cpp` / `screen.h`
- `src/emu/emuopts.cpp` / `emuopts.h`

---

### BUILD 2D OPTIMISÉE — NETTOYAGE ARCADE.LST

Version allégée spécialisée pour l'arcade 2D classique sur CRT 15 kHz.

#### Systèmes supprimés :

- ✗ Tous les jeux 3D (Model 2/3, Naomi, Taito Type X etc.)
- ✗ Systèmes casino (machines à sous, poker vidéo)
- ✗ Jeux de mahjong
- ✗ Systèmes "machine" non-arcade (ordinateurs, consoles)
- ✗ Systèmes jeux mécaniques etc.

#### Systèmes conservés (2D uniquement) :

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

L'appel périodique Lua periodic_check et frame_hook est maintenant commenté pour éviter tout impact sur les performances ou les menus.

---

### HARDCADE SWITCHRES - CRT-OPTIMIZED MODE SELECTION

#### Objectif :

Amélioration du switchres MAME pour privilégier le refresh exact sur CRT.

#### Problème MAME original :

Privilégie la résolution exacte, puis le refresh.  
→ Sur CRT, ±0,01 Hz de refresh = scrolling saccadé

#### Solution HARDCADE :

Inverser la priorité : **Refresh > Height > Width**

#### Scoring modifié :

- Refresh score = 1000 / (1 + diff_refresh) ← **Priorité absolue**
- Height match = +200
- Width ≥ jeu = +50
- Width > +16px = -20
- Rejection si diff_refresh > 1 Hz

#### Exemple :

Jeu Neo Geo : 304x224 @ 59.185 Hz

**Modelines :**
- `320x224@60` → score: 801 (diff: 0.815 Hz)
- `321x224@59` → score: 1094 (diff: 0.185 Hz) ✅ **CHOISI**

**Résultat :** MAME choisit toujours le refresh le plus proche, même si la résolution diffère légèrement.

✓ Fonctionne avec CRT_emudriver / Soft15khz / PowerStrip

**Fichier modifié :** `src/osd/modules/render/drawdd.cpp` (enum_modes_callback)

---

## COMPATIBILITÉ

| Type | Support |
|------|---------|
| **OS** | Windows XP / 7 / 8 / 10 |
| **Renderers** | DDraw (XP), D3D9, GDI |
| **Monitors** | CRT 15 kHz, LCD 31 kHz |

---

## CRÉDITS

**Développement :** Olivier Mileo  
**Projet :** CRT-MAME ARCADE-2D 0.168 Edition  
**Base :** MAME 0.168

© 2025 Hardcade — Tous droits réservés
