# Preuves vidéo appariées — point 4 de la review de travle1

Projet : `multisheet-styled.pc` — deux feuilles (`Main`, `Power`), un fil par feuille reliés par un
label global `BUS`, **aucun symbole** (donc aucun badge orange d'empilement). Le fil stylé est
visible sur `Main` et l'apparence s'applique aux deux feuilles.

| Fichier | Contenu |
|---|---|
| `replay-multisheet-after.mkv` | Projet stylé (couleur accent `#3a88ee`, largeur 5 pt) : zoom avant, zoom arrière, drag du fil, puis Undo. 9,6 s, 1720×1223, 48 images. |
| `replay-multisheet-before.mkv` | **Même projet**, apparence remise au thème (Reset to theme default), mêmes actions dans le même ordre : zoom avant/arrière, drag du fil, Undo. 14,3 s, 1720×1279, 37 images. |
| `64-fixture-loaded.png` / `67-multisheet-main.png` | Ouverture du fixture et fil stylé sur `Main` (aucun badge de symbole). |
| `68-wire-selected-multisheet.png` | Fil sélectionné avec le cue contrasté. |
| `69-reset-dialog.png` / `70-after-reset.png` | Formulaire Net appearance et retour à l'apparence du thème pour le « before ». |

Conditions appariées : même onglet Chrome, même viewport, mêmes coordonnées de zoom et de drag,
même séquence d'actions. Les vidéos sont en Matroska MJPEG (aucun ffmpeg nécessaire ; lisibles avec
VLC/mpv). Elles ont été capturées par CDP (`screenshotFallback: true`), donc le nombre d'images
effectif est inférieur au débit nominal.

Limite : le drag est suivi d'un Undo pour ne pas modifier la géométrie entre les deux prises ; le
replay montre donc l'aperçu de drag, pas un déplacement commité.
