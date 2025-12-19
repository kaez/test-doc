# Installation

## Prérequis

Avant de commencer l'installation, assurez-vous d'avoir :

- Node.js version 16 ou supérieure
- npm ou yarn installé
- Git configuré sur votre machine

## Installation via npm

```bash
npm install -g mon-application
```

## Installation via yarn

```bash
yarn global add mon-application
```

## Vérification de l'installation

Pour vérifier que l'installation s'est bien déroulée :

```bash
mon-application --version
```

Vous devriez voir s'afficher le numéro de version actuel.

## Installation depuis les sources

Si vous préférez installer depuis les sources :

```bash
git clone https://github.com/example/mon-application.git
cd mon-application
npm install
npm run build
npm link
```

## Problèmes courants

### Erreur de permissions

Si vous rencontrez une erreur de permissions sous Linux/Mac :

```bash
sudo npm install -g mon-application
```

### Version de Node.js incompatible

Assurez-vous d'utiliser une version compatible de Node.js :

```bash
node --version
# Doit afficher v16.x.x ou supérieur
```

## Prochaines étapes

Une fois l'installation terminée, consultez le [guide de configuration](configuration.md).
