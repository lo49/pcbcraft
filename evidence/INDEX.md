# Captures visuelles — PR #478

App servie depuis server2 : `http://192.168.31.215:8781/` (build WASM de la branche `fix/review-travle1`).
Scénario construit dans l'UI : deux symboles GND reliés par un fil, puis **Net appearance…** :
couleur = accent du thème sombre (`#3a88ee`, RGB 58/136/238) et largeur maximale (800 = 8,00 pt).
C'est exactement le cas pathologique demandé par `travle1` au point 2.

| Fichier | Contenu |
|---|---|
| `39-applied-selected2.png` | Thème sombre : fil stylé couleur accent, 8 pt, non sélectionné. |
| `40-selected-accent-cue.png` | Thème sombre : le même fil sélectionné — le liseré clair reste visible sous l'encre accent. |
| `41-wire-clip.png` | Zoom sur le fil sélectionné (360×40) : underlay clair de part et d'autre de l'ink. |
| `43-light-theme-selected.png` | Thème clair : le fil sélectionné avec son cue. |
| `44-light-theme-clip.png` | Zoom sur le fil en thème clair. |
| `45-zoom-in.png` | Thème clair, zoom avant : le cue reste lisible. |
| `46-zoom-out.png` | Thème clair, zoom arrière : idem. |
| `replay.mkv` | Vidéo 8,7 s (1920×863, 20 i/s, 63 images) : déplacement caméra (zoom avant/arrière) puis aperçu de drag sur le fil stylé. Format Matroska MJPEG (aucun ffmpeg requis ; lisible avec VLC/mpv/ffmpeg). |

Les autres images (`01` à `38`) documentent la construction du scénario : chargement de l'app,
création du schéma, pose des symboles, câblage, ouverture du formulaire **Net appearance**,
réglage de la largeur et de la couleur.

Limitations : le replay est sur un projet à une seule feuille (le feature cross-sheet est couvert
par le test headless `net_appearance_scope_follows_shared_net_not_same_named_local_nets`) ; la
vidéo a été produite par capture CDP de l'onglet Chrome (`screenshotFallback: true`, 8 i/s effectifs),
pas par une sortie GPU native.
