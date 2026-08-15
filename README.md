# Mon Eau : l'eau autour de chez soi, pour Home Assistant

[![HACS Custom](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)
[![Validate](https://github.com/steph9742/ha-mon-eau/actions/workflows/validate.yml/badge.svg)](https://github.com/steph9742/ha-mon-eau/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Intégration Home Assistant qui rassemble **toutes les données publiques françaises sur l'eau autour de chez vous** :

- **Eau du robinet** : conformité du contrôle sanitaire (bulletins ARS + Hub'eau), nitrates, pesticides, bactériologie, par réseau de distribution
- **Rivière** : hauteur et débit **temps réel** (Hub'eau hydrométrie), vigilance crues et prévisions (Vigicrues), santé de la rivière (température, oxygène, pH : Hub'eau qualité des cours d'eau)
- **Nappe phréatique** : niveau piézométrique, souvent en temps réel horaire (Hub'eau)
- **Sécheresse** : niveau de restriction et usages interdits (VigiEau), assecs des petits cours d'eau (ONDE)
- **Baignade** : qualité du site de baignade ARS le plus proche (lac, rivière, mer), avec dernier contrôle, bactériologie et classement européen (saisonnier)

**5 cartes Lovelace dédiées sont incluses** : aucune installation séparée. Sources : APIs publiques, sans clé.

> *English: Home Assistant integration gathering French public water data around your home: tap water sanitary compliance (ARS/Hub'eau), real-time river level and flow with flood warnings (Vigicrues), groundwater level, and drought restrictions (VigiEau). Ships with 5 dedicated Lovelace cards. See [English section](#english) below.*

## Philosophie

Les fraîcheurs de ces données vont de **10 minutes** (hydrométrie) à **plusieurs mois** (qualité des rivières). L'intégration ne laisse jamais croire à du temps réel : chaque capteur porte la **date de sa mesure**, et les cartes l'affichent systématiquement.

## Installation

### Via HACS (recommandé)

1. Dans HACS, menu **⋮** → **Dépôts personnalisés**
2. Ajoutez `https://github.com/steph9742/ha-mon-eau` avec le type **Intégration**
3. Installez **Mon Eau**, redémarrez Home Assistant
4. **Paramètres → Appareils et services → Ajouter une intégration** → **Mon Eau**

### Manuellement

Copiez `custom_components/mon_eau` dans votre dossier `custom_components`, redémarrez.

> Le logo de l'intégration est embarqué (`brand/`) et s'affiche automatiquement à partir de **Home Assistant 2026.3** (API brands locale). Sur les versions antérieures, l'icône par défaut est affichée.

## Configuration

Tout se fait dans l'interface :

1. **Plateformes** : cochez ce qui vous intéresse (eau potable, rivière, nappe, sécheresse, baignade)
2. **Commune** : recherche par nom (via geo.api.gouv.fr)
3. **Réseaux de distribution** : une commune peut avoir plusieurs réseaux aux résultats différents ; vous pouvez en suivre **plusieurs à la fois** (un appareil chacun), et choisir un réseau précis permet d'utiliser le bulletin ARS, plus frais d'environ un mois que Hub'eau
4. **Stations** : hydrométrie, piézomètres et sites de baignade acceptent la **sélection multiple** (un appareil par station) ; les listes sont **triées par distance** et **filtrées sur la fraîcheur réelle des données** (beaucoup de stations publiques sont à l'arrêt)

## Entités créées

| Plateforme | Entités | Fraîcheur | Polling |
|---|---|---|---|
| Eau potable | `binary_sensor` conformité (problème), dernier prélèvement, nitrates, pesticides, E. coli, entérocoques | ~2 prélèvements/sem., publication 2-3 sem. (ARS) à 6-7 sem. (Hub'eau) | 12 h |
| Rivière | hauteur (m), débit (m³/s), **situation hydrologique** (vs normales de saison sur 15 ans), tendances de hauteur 24 h / 7 j / 15 j, vigilance crues, température, oxygène, pH, nitrates | 10 min (hydro) ; 1-2 mois + retard (santé) | 15 min / 24 h |
| Nappe | profondeur (m), niveau NGF (m), **situation de la nappe** (vs normales saisonnières), **remplissage** (% de la plage historique), tendance (enum), tendances 24 h / 7 j / 15 j | horaire (si télétransmis) | 1 h |
| Sécheresse | niveau de gravité (enum), `binary_sensor` restrictions | arrêtés préfectoraux | 6 h |
| Baignade | `binary_sensor` eau de baignade, classement européen, dernier contrôle (avec E. coli / entérocoques et seuils) | contrôles ARS en saison | 6 h (24 h hors saison) |

Les valeurs numériques ont `state_class: measurement` → statistiques long terme utilisables dans les graphiques HA.

## Cartes Lovelace incluses

Les 5 cartes apparaissent dans le sélecteur de cartes (avec éditeur visuel). Configuration minimale :

```yaml
type: custom:mon-eau-water-card
device: <choisi dans l'éditeur>
# compact: true   # variante une ligne
```

L'éditeur visuel ne propose que les appareils du bon type, et chaque carte a ses options d'affichage (barres de paramètres, conclusion ARS, courbe, tendances, santé de la rivière, usages, historique…).

- **`mon-eau-water-card`** : pastille conforme/non conforme, paramètres en % de la limite réglementaire (fournie par l'API), conclusion ARS dépliable. Bordure rouge et conclusion auto-dépliée si non conforme ; état grisé si données anciennes.
- **`mon-eau-river-card`** : hauteur + débit en tuiles avec tendance, sparkline 24 h, valeurs « santé » datées. **Mode crue automatique** : badge et bordure à la couleur de vigilance Vigicrues, courbe de hauteur prolongée par la **prévision en pointillés**.
- **`mon-eau-groundwater-card`** : profondeur + niveau NGF, tendance et courbe 30 jours.
- **`mon-eau-drought-card`** : jauge 4 niveaux (vigilance → crise), gravité par ressource (nappe / rivières / eau potable), usages restreints, dates d'arrêté + PDF, assecs observés.
- **`mon-eau-bathing-card`** : carte saisonnière à **repli automatique** : en saison, verdict du dernier contrôle, bactériologie avec seuils réglementaires, classement et historique ; hors saison, une ligne discrète avec la date de réouverture.

## Sources de données

| Source | Usage | Accès |
|---|---|---|
| [Hub'eau](https://hubeau.eaufrance.fr) | eau potable, hydrométrie, température, qualité rivières, nappes, ONDE | API JSON sans clé |
| [orobnat.sante.gouv.fr](https://orobnat.sante.gouv.fr) | bulletins ARS récents (eau potable) | lecture du site officiel, repli automatique sur Hub'eau |
| [VigiEau](https://vigieau.gouv.fr) | restrictions sécheresse | API JSON sans clé |
| [Vigicrues](https://www.vigicrues.gouv.fr) | vigilance et prévisions de crues | API JSON sans clé |
| [baignades.sante.gouv.fr](https://baignades.sante.gouv.fr) | qualité des eaux de baignade (ARS) | lecture du site officiel |

## Limitations connues

- Les données « santé rivière » (température, O₂, pH) sont des prélèvements ponctuels publiés avec plusieurs mois de retard : c'est la donnée publique disponible, les cartes affichent la date.
- Le réseau de stations de **température en continu** est presque entièrement à l'arrêt ; l'étape de configuration ne propose que les stations réellement vivantes (rares).
- Le bulletin ARS est lu depuis le site officiel (pas d'API) : en cas de changement du site, l'intégration se replie silencieusement sur Hub'eau.

---

## English

**Mon Eau** ("My Water") gathers French public water data around your home into Home Assistant: tap water sanitary compliance (ARS bulletins + Hub'eau, per distribution network), real-time river level/flow with Vigicrues flood warnings and forecasts, river health (temperature, oxygen, pH), groundwater level (often hourly), VigiEau drought restrictions, and seasonal bathing water quality (ARS controls with bacteriology). Five dedicated Lovelace cards are bundled: no separate install. All sources are keyless public APIs; every sensor carries its measurement date, because data freshness ranges from 10 minutes to several months.

Setup is UI-only: pick platforms, search your municipality, pick your distribution network and nearby stations (lists are sorted by distance and filtered to stations with actually fresh data). France only.

## Licence

MIT : voir [LICENSE](LICENSE). Non affilié aux services publics cités.
