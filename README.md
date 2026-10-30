# ArcGIS Layer Finder

Remonte les endpoints de geoserveur (ArcGIS REST, WMS, WFS, WMTS, WCS...) exploitables dans QGIS à partir d'une application web cartographique.

## Fonctionnement

**Mode ArcGIS** : colle l'URL d'une app ArcGIS Online (Instant Apps, StoryMaps, Dashboards, webmap...) ou directement un ID d'item. L'outil interroge la sharing REST API ArcGIS (`www.arcgis.com/sharing/rest/content/items/...`), lit la config (`data`) de l'item, suit récursivement les références (`webmap`/`webscene`/`itemId`/`mapId`, profondeur max 3) et collecte toutes les URLs de services trouvées dans la config.

**Mode Ma Carte IGN** : colle l'URL d'une carte `macarte.ign.fr/carte/<id>/...`. L'outil interroge l'API publique de Ma Carte (`macarte.ign.fr/api/maps/<id>` et `.../file`, qui répond en CORS ouvert), lit la config des couches et en extrait automatiquement les URLs de services (WMS, WFS...) ainsi que les codes de couches Géoplateforme IGN natives (type `Geoportail`), reconstruits en URL WMTS (`data.geopf.fr/wmts`).

**Mode fetch direct** : pour toute autre URL, l'outil tente de récupérer la page telle quelle (`fetch`) et d'en scanner le texte avec les mêmes motifs. Ça marche quand le site autorise le CORS sur ses pages/fichiers statiques (rare mais ça arrive) ; sinon le navigateur bloque la lecture et un message invite à utiliser le mode texte brut ci-dessous.

**Mode texte brut** : pour tout autre site (GeoServer open-source, QGIS Server auto-hébergé, MapServer CGI/FastCGI, ou une app dont les services n'apparaissent que dans le trafic réseau d'une SPA et pas dans le HTML statique), colle le code source (Ctrl+U), un bundle JS, ou des URLs copiées depuis l'onglet réseau du navigateur dans le second champ et clique "Scanner le texte". Aucune requête réseau n'est faite : l'outil extrait toutes les URLs du texte collé et ne garde que celles qui ressemblent à un service de geoserveur. Ce mode fonctionne même quand le site bloque le CORS, puisqu'il n'y a pas de fetch.

## Formats de geoserveur détectés

- **Esri REST** : MapServer, FeatureServer, ImageServer, SceneServer, VectorTileServer, GeocodeServer
- **OGC** : WMS, WFS, WMTS, WCS — via un chemin explicite (`/wms`, `/geoserver/...`, `/ows`...), via `?service=WMS|WFS|WMTS|WCS` en query string, ou via un endpoint CGI/FastCGI type `mapserv.fcgi`/`qgis_mapserv.fcgi`
- **Chemins relatifs** : une config qui référence son service par un chemin sans domaine (ex. `"geocodeService": "/arcgis/rest/services/.../GeocodeServer"`, fréquent quand l'app et son ArcGIS Server/GeoServer partagent la même origine) est aussi détectée, en mode texte brut comme en mode ArcGIS/Ma Carte IGN. En mode fetch direct, ces chemins sont automatiquement résolus en URL absolue puisque l'origine de la page est connue ; en mode texte brut, l'origine n'est pas connue donc le chemin est affiché tel quel avec une note "à préfixer avec le domaine de l'app/API"

## Usage

Ouvrir `index.html` dans un navigateur, ou servir en local :

```
python3 -m http.server 8000
```

puis aller sur `http://localhost:8000/index.html`.

Entrées acceptées : URL `macarte.ign.fr/carte/<id>/...`, URL ArcGIS complète avec `?appid=...`, URL `items/<id>`, ou ID brut ArcGIS (32 caractères hex).

## Résultat

- Une carte par item/projet traversé (titre, auteur/propriétaire, date, description)
- Un tableau des services trouvés, avec le nombre de couches/tables détectées pour les services Esri consultables
- Un bouton "Copier" par ligne et un bouton "Copier tout" pour récupérer toutes les URLs d'un coup (à coller dans QGIS, un fichier de notes...)

## Limites connues

- Le mode ArcGIS fonctionne uniquement contre ArcGIS Online — pas les portails ArcGIS Enterprise sur domaine propre
- Les items ArcGIS privés échouent (pas d'authentification gérée)
- Un service protégé par CORS ou non-Esri s'affiche comme "non consultable" dans la colonne détails
- La traversée des références ArcGIS est plafonnée à une profondeur de 3
- Le mode Ma Carte IGN ne suit pas de références récursives (un seul projet à la fois) et ne couvre que les API publiques connues de macarte.ign.fr — pas les autres plateformes SPA (celles-là passent par le mode texte brut, avec les URLs copiées depuis l'onglet réseau du navigateur si le HTML seul ne les contient pas)
- Le mode fetch direct échoue sur la plupart des sites (pas de CORS sur du HTML/JS classique) — c'est une tentative, pas une garantie, et il ne décrit pas les couches, il extrait juste des URLs
- Le mode texte brut ne fait aucune requête réseau : il extrait des URLs, il ne décrit pas les couches

## Pistes d'amélioration non implémentées

- URL de portail configurable (support ArcGIS Enterprise)
- Authentification pour les items privés
- Export JSON/CSV des résultats
- Timeout sur les appels réseau
- Détection des sous-dossiers/sous-couches de services
- Support d'autres plateformes à API ouverte en CORS (sur le même principe que Ma Carte IGN)

## Contexte d'usage

Outil de reconnaissance/OSINT pour analyser des apps cartographiques publiques — à utiliser sur des cibles autorisées.
