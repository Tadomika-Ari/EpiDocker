# EpiDocker

Image Docker Alpine preconfiguree pour disposer rapidement de l'ecosysteme Epiclang:

- Epiclang (depuis Epifaster)
- Criterion (framework de tests C)
- Outils de build C/C++ modernes (clang, cmake, meson, ninja, make)

Le but du projet est simple: fournir un environnement reproductible pour compiler, tester et executer des projets C avec Epiclang, sans configurer manuellement chaque dependance.

Il est aussi pensé pour l'optimisation de temps, notamment dans les pipelines GitHub Actions, en evitant de refaire toute la configuration d'outils a chaque setup CI.

## Apercu

Ce projet construit une image qui:

1. Part de `alpine`.
2. Installe la toolchain necessaire (clang20, make, cmake, meson, ninja, etc.).
3. Clone et installe Criterion.
4. Clone Epifaster et installe `epiclang` dans le systeme.
5. Nettoie les dependances de build pour reduire la taille finale de l'image.

## Prerequis

- Docker installe sur ta machine

## Build de l'image

Depuis la racine du projet:

```bash
docker build -t epidocker:latest .
```

## Utilisation rapide

Lancer un shell interactif dans le conteneur:

```bash
docker run --rm -it epidocker:latest sh
```

Verifier les outils principaux:

```bash
epiclang --help
clang --version
```

Monter un projet local dans le conteneur:

```bash
docker run --rm -it \
	-v "$(pwd)":/workspace \
	-w /workspace \
	epidocker:latest sh
```

## Optimisation pour GitHub Actions

Cette image est particulierement utile en CI pour reduire le temps de preparation des jobs.
En centralisant les dependances de build dans une image Docker prete a l'emploi, tu limites les etapes repetitives d'installation dans GitHub Actions et tu rends les executions plus stables.

## Ce que contient l'image

- Base: Alpine Linux
- Compilation: clang20, clang20-rtlib, build-base, make
- Build system: cmake, meson, ninja
- Outils annexes: gcovr, libffi-dev, libgit2-dev
- Tests: Criterion compile et installe
- Langage: `epiclang` installe dans `/usr/bin/epiclang`

## Structure du projet

```text
.
├── Dockerfile
└── README.md
```

## Exemple de workflow

1. Construire l'image.
2. Monter ton dossier de projet avec `-v`.
3. Compiler et tester dans le conteneur.

Exemple:

```bash
docker build -t epidocker:latest .
docker run --rm -it -v "$(pwd)":/workspace -w /workspace epidocker:latest sh
# puis dans le conteneur:
# make
# epiclang ...
```

## Depannage

- `epiclang: not found`
	- Verifie que tu lances bien une image construite a partir de ce Dockerfile.
	- Teste `ls -l /usr/bin/epiclang` dans le conteneur.

- Probleme reseau pendant le build (clone GitHub)
	- Relance le build:

```bash
docker build --no-cache -t epidocker:latest .
```

- Permission denied sur les fichiers montes
	- Lance le conteneur avec un utilisateur compatible avec ton host si necessaire.

## Personnalisation

Tu peux adapter facilement l'image en modifiant [Dockerfile](Dockerfile):

- Ajouter des paquets Alpine supplementaires.
- Epingler des commits/tags precis pour Epifaster et Criterion.
- Ajouter un `ENTRYPOINT` selon ton workflow.

## Licence

Ce depot ne fournit que la configuration Docker de l'environnement.
Les licences des projets tiers (Epifaster, Criterion, Alpine et dependances) s'appliquent a leurs composants respectifs.
