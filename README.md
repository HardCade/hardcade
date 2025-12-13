## 📥 Téléchargement

Voir les [Releases](../../releases) pour télécharger.


README – CRT-MAME 0.168 V1.3

Version Développement – Décembre 2025
© 2025 Hardcade – PlayRetro – Olivier Mileo

================================================================================
DÉVELOPPÉ PAR : Olivier Mileo
PROJET : Edition CRT-MAME 0.168
COPYRIGHT : © 2025 Hardcade – PlayRetro – Tous droits réservés
DESCRIPTION

CRT-MAME 0.168 est une version spécialisée de MAME 0.168 optimisée pour :

les écrans CRT 15 kHz (arcade, TV, PVM/BVM, Hantarex, Nanao, etc.) et les écrans LCD 

un input lag minimal

un contrôle avancé et précis du refresh rate par jeu

Objectif : 

    - Obtenir un input lag minimal
    - Obtenir des performances d'affichage accrues et un comportement supérieur (aux émulateurs déjà existants) pour le CRT 15 kHz. 
    - Obtenir un affichage ultra smooth et des performances optimales sur écran LCD.

================================
PRINCIPALES FONCTIONS
================================

- Sauvegarde du slider Refresh Rate dans les fichiers CFG

- Amélioration du slider (précision, stabilité, arrondi, gestion du défaut)

- Ajout du timestamp input Windows (préparation anti-lag)

- Ajout du Late Input Polling (réduction d’1 frame complète d’input lag)

- Ajout optimisation Direct3D – “Direct Present Minimal” (permettant une réduction du lag d’affichage)

- Ajout HardSync Adaptatif (Stabilise le timing frame et évite les oscillations, réduit la latence globale)


=============================================================
LISTE DES FICHIERS MODIFIES
=============================================================

src/emu/screen.cpp
src/emu/screen.h
src/emu/ui/ui.cpp
src/emu/video.cpp
src/osd/windows/input.cpp
src/osd/windows/video.cpp
src/osd/modules/render/d3d/d3d9intf.cpp


=============================================================
CRT-MAME 0.168 — V1.3
Notes de modification
=============================================================
1) SLIDER REFRESH RATE CFG — Sauvegarde initiale (V1)

Fonctionnalité :
Sauvegarde et rechargement automatique de la fréquence utilisateur dans :
cfg/[nom_du_jeu].cfg

Fichiers modifiés :

src/emu/video.cpp → config_save_screen_refresh() / config_load_screen_refresh()

-------------------------------------------------------------------------------------------------------------------------------

2) SLIDER REFRESH RATE — Améliorations (V2)

Améliorations :

Slider dans le menu TAB fonctionnel et stable

Arrondi à 3 décimales pour cohérence exacte

Sauvegarde uniquement si différence réelle

Ajout de la fréquence d’origine pour comparaison propre

Application immédiate de la nouvelle fréquence

Fichiers modifiés :

src/emu/ui/ui.cpp — slider_refresh()

src/emu/screen.h — m_user_refresh_rate, setters/getters, valeur par défaut

src/emu/screen.cpp — initialisation default refresh

src/emu/video.cpp — V2 CFG load/save


NOUVELLE FONCTION AJOUTEE dans src/emu/video.cpp :

//-------------------------------------------------
//  HARDCADE config_load_screen_refresh - load screen refresh rates MODIF
//-------------------------------------------------

static void config_load_screen_refresh(running_machine &machine, int cfg_type, xml_data_node *parentnode)
{
	// only load game configurations
	if (cfg_type != CONFIG_TYPE_GAME || parentnode == NULL)
		return;

	// iterate over screen nodes
	for (xml_data_node *screennode = xml_get_sibling(parentnode->child, "screen"); 
	     screennode != NULL; 
	     screennode = xml_get_sibling(screennode->next, "screen"))
	{
		// get the screen tag
		const char *screen_tag = xml_get_attribute_string(screennode, "tag", "");
		if (screen_tag[0] == 0)
			continue;
		
		// find the matching screen device
		screen_device_iterator iter(machine.root_device());
		for (screen_device *screen = iter.first(); screen != NULL; screen = iter.next())
		{
			if (strcmp(screen->tag(), screen_tag) == 0)
			{
				// load and apply the refresh rate
				double refresh = xml_get_attribute_float(screennode, "refresh", 0.0);
				if (refresh > 0.0)
				{
					// store the user refresh rate
					screen->set_user_refresh_rate(refresh);
					
					// apply it immediately
					int width = screen->width();
					int height = screen->height();
					const rectangle &visarea = screen->visible_area();
					screen->configure(width, height, visarea, HZ_TO_ATTOSECONDS(refresh));
				}
				break;
			}
		}
	}
}

//-------------------------------------------------
//  HARDCADE config_save_screen_refresh - save screen refresh rates
//-------------------------------------------------

static void config_save_screen_refresh(running_machine &machine, int cfg_type, xml_data_node *parentnode)
{
	// only save game configurations
	if (cfg_type != CONFIG_TYPE_GAME || parentnode == NULL)
		return;

	// iterate over all screens
	screen_device_iterator iter(machine.root_device());
	for (screen_device *screen = iter.first(); screen != NULL; screen = iter.next())
	{
		// only save if user has set a custom refresh rate
		double refresh = screen->user_refresh_rate();
		if (refresh > 0.0)
		{
			// create a screen node
			xml_data_node *screennode = xml_add_child(parentnode, "screen", NULL);
			if (screennode != NULL)
			{
				xml_set_attribute(screennode, "tag", screen->tag());
				xml_set_attribute_float(screennode, "refresh", refresh);
			}
		}
	}
}


FONCTION MODIFIEE DANS dans src/emu/ui/ui.cpp :

//-------------------------------------------------
//  HARDCADE slider_refresh - refresh rate slider callback
//-------------------------------------------------

static INT32 slider_refresh(running_machine &machine, void *arg, std::string *str, INT32 newval)
{
    screen_device *screen = reinterpret_cast<screen_device *>(arg);
    double defrefresh = screen->default_refresh_rate();
    double refresh;

    if (newval != SLIDER_NOCHANGE)
    {
        int width = screen->width();
        int height = screen->height();
        const rectangle &visarea = screen->visible_area();
        
        // nouvelle valeur brute
        double new_refresh = defrefresh + (double)newval * 0.001;
        
        // APPLIQUER la valeur (imprécise, c’est normal)
        screen->configure(width, height, visarea, HZ_TO_ATTOSECONDS(new_refresh));

        // ARRONDI pour l'affichage et la sauvegarde
        double rounded_new = floor(new_refresh * 1000.0 + 0.5) / 1000.0;
        double rounded_default = floor(defrefresh * 1000.0 + 0.5) / 1000.0;

        // SAUVEGARDER uniquement la valeur arrondie
        if (rounded_new != rounded_default)
            screen->set_user_refresh_rate(rounded_new);  // <-- CORRECT !
        else
            screen->set_user_refresh_rate(0.0);
    }

	if (str != NULL)
		strprintf(*str, "%.3ffps", ATTOSECONDS_TO_HZ(screen->frame_period().attoseconds()));

	refresh = ATTOSECONDS_TO_HZ(screen->frame_period().attoseconds());
	return floor((refresh - defrefresh) * 1000.0 + 0.5);
}

-------------------------------------------------------------------------------------------------------------------------------


3) INPUT LAG — Timestamp Windows (V3)

Objectif : capturer l’instant exact où un input est mis à jour.

Fonctionnalité :
Ajout d’un timestamp haute précision lors des mises à jour :

Win32 keyboard

RawInput

DirectInput (joysticks USB + HID anciens)

Fichiers modifiés :

src/osd/windows/input.cpp

ajout UINT64 last_timestamp dans class device_info

ajout de devinfo->last_timestamp = osd_ticks(); dans :

win32_keyboard_poll()

rawinput_keyboard_update()

dinput_joystick_poll()

-------------------------------------------------------------------------------------------------------------------------------

4) INPUT LAG — Late Input Polling (V3b)

Nouvelle fonctionnalité majeure : Ajout du Late Input Polling (LIP), réduction d’1 frame d’input lag.

   - Polling des périphériques déplacé juste avant le traitement d’input
   - Permet de réduire l’input lag d’une frame complète
   - Fonction ajoutée : late_input_poll()
   - Intégrée dans windows_osd_interface::update()


Principe :
Forcer un polling ultra tardif des périphériques juste avant que Windows traite les messages de la frame → input plus frais = 1 frame gagnée.

Ajouts :

a) Nouvelle fonction

src/osd/windows/input.cpp :

void late_input_poll(running_machine &machine)
{
    osd_lock_acquire(input_lock);

    if (keyboard_list)   device_list_poll_devices(keyboard_list);
    if (mouse_list)      device_list_poll_devices(mouse_list);
    if (lightgun_list)   device_list_poll_devices(lightgun_list);
    if (joystick_list)   device_list_poll_devices(joystick_list);

    last_poll = GetTickCount();

    osd_lock_release(input_lock);
}

b) Appel dans la boucle Windows (critique)

Dans src/osd/windows/video.cpp, fonction :

	Insertion de late_input_poll(machine()) dans windows_osd_interface::update()
	Ajout du extern void late_input_poll(running_machine &machine);

-------------------------------------------------------------------------------------------------------------------------------

5) Optimisation Direct3D – “Direct Present Minimal”

Une optimisation GPU inspirée de GroovyMAME permettant une réduction du lag d’affichage.

- Fonctionnalités ajoutées

Présentation d’image immédiate via SwapChain->Present()

Contournement du buffering interne Direct3D

Réduction du lag GPU d’environ 1 frame

Compatible Windows XP / Direct3D 9

Fallback sécurisé si l’option n’est pas supportée par le driver

- Fichiers modifiés :

src/osd/modules/render/d3d/d3d9intf.cpp

- Remplacement complet de la fonction :

device_present()

// HARDCADE MODIFICATION Direct Present Minimal ///////////////////////////////////
static HRESULT device_present(device *dev, const RECT *source, const RECT *dest, HWND override, RGNDATA *dirty, DWORD flags)
{
    IDirect3DDevice9 *device = (IDirect3DDevice9 *)dev;

    // If flags are provided, prefer presenting via the swapchain with those flags
    if (flags != 0)
    {
        IDirect3DSwapChain9 *chain = NULL;
        HRESULT result = IDirect3DDevice9_GetSwapChain(device, 0, &chain);
        if (result == D3D_OK && chain != NULL)
        {
            result = IDirect3DSwapChain9_Present(chain, source, dest, override, dirty, flags);
            IDirect3DSwapChain9_Release(chain);
            return result;
        }
    }

    // Attempt to present via the swapchain with FORCEIMMEDIATE if available.
    // This is preferred because IDirect3DDevice9::Present() has no flags parameter.
    IDirect3DSwapChain9 *chain = NULL;
    HRESULT r = IDirect3DDevice9_GetSwapChain(device, 0, &chain);
    if (r == D3D_OK && chain != NULL)
    {
#ifdef D3DPRESENT_FORCEIMMEDIATE
        HRESULT pres = IDirect3DSwapChain9_Present(chain, source, dest, override, dirty, D3DPRESENT_FORCEIMMEDIATE);
#else
        // Fall back to zero flags if FORCEIMMEDIATE is not defined
        HRESULT pres = IDirect3DSwapChain9_Present(chain, source, dest, override, dirty, 0);
#endif
        IDirect3DSwapChain9_Release(chain);
        return pres;
    }

    // Last-resort fallback: call device Present (no flags)
    return IDirect3DDevice9_Present(device, source, dest, override, dirty);
}

// HARDCADE Direct Present Minimal ///////////////////////////////////


- Description rapide

Le patch force Direct3D à présenter l’image dès que possible, sans passer par la file d’attente GPU habituelle.
Résultat : affichage plus direct, image plus proche du timing réel, meilleure réactivité en 15 kHz et LCD.

- Interaction avec les autres optimisations

Cette optimisation s’ajoute à :

Sauvegarde personnalisée du refresh rate

Late Input Polling

Amélioration du système d’entrée DirectInput / RawInput

Aucune incompatibilité connue.

Configuration ini recommandée pour bénéficier de l’amélioration GPU :

-video d3d
-waitvsync 0

-------------------------------------------------------------------------------------------------------------------------------

6) HARDCADE CRT-MAME — HardSync Adaptatif

Ajout d’un système léger de synchronisation CPU avant le rendu vidéo, compatible Windows XP et petites configurations.

Effet :

Réduit les micro-saccades (micro-stutter)

Évite les Present() trop précoces

Aucune charge CPU (Sleep(0))

Fonctionne avec : DirectDraw, Direct3D, GDI

- Modifications appliquées
Fichier modifié : src/osd/windows/video.cpp

Dans la fonction :

void windows_osd_interface::update(bool skip_redraw)

Un nouveau bloc a été ajouté avant le Late Input Poll :

// HARDCADE CRT-MAME : HardSync Adaptatif (latence réduite)

if (video_config.waitvsync == FALSE && video_config.syncrefresh == FALSE)
{
    DWORD now = timeGetTime();
    DWORD delta = now - last_event_check;

    if (delta < 10)
    {
        Sleep(0); // yield CPU → stabilise le timing
    }
}

- Objectif du HardSync Adaptatif

Stabiliser le timing frame et éviter les oscillations

Réduire la latence globale sans bloquer la machine

Fonctionne même lorsque la carte vidéo est ancienne ou lente

Version simplifiée et plus sûre qu’un vrai Hard Sync GroovyMAME


================================
VERSION
================================

CRT-MAME 0.168 — V1.3
Décembre 2025
© 2025 Hardcade – PlayRetro – Olivier Mileo
================================================================================

