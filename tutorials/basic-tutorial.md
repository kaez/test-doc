# Tutoriel de base

## Introduction

Ce tutoriel vous guidera à travers les concepts de base de l'application. À la fin de ce tutoriel, vous saurez :

- Utiliser les fonctionnalités principales
- Manipuler les données
- Gérer les erreurs
- Optimiser vos workflows

## Durée estimée

⏱️ Environ 30 minutes

## Partie 1 : Les bases

### Créer une instance

Commençons par créer une instance de base :

```javascript
const { Client } = require('mon-application');

const client = new Client({
  apiKey: process.env.API_KEY,
  debug: true
});
```

### Effectuer une requête simple

```javascript
async function exempleSimple() {
  try {
    const result = await client.get('/data');
    console.log('Résultat:', result);
  } catch (error) {
    console.error('Erreur:', error.message);
  }
}

exempleSimple();
```

## Partie 2 : Manipulation de données

### Créer des données

```javascript
async function creerDonnees() {
  const nouvelleDonnee = await client.create({
    nom: 'Exemple',
    description: 'Ceci est un exemple',
    tags: ['test', 'demo'],
    metadata: {
      createdBy: 'tutoriel',
      timestamp: Date.now()
    }
  });

  console.log('Créé avec l\'ID:', nouvelleDonnee.id);
  return nouvelleDonnee;
}
```

### Lire des données

```javascript
async function lireDonnees(id) {
  const donnee = await client.findById(id);

  console.log('Données trouvées:', {
    id: donnee.id,
    nom: donnee.nom,
    dateCreation: new Date(donnee.createdAt)
  });

  return donnee;
}
```

### Mettre à jour des données

```javascript
async function mettreAJourDonnees(id, modifications) {
  const donneeMAJ = await client.update(id, modifications);
  console.log('Mise à jour effectuée:', donneeMAJ);
  return donneeMAJ;
}
```

### Supprimer des données

```javascript
async function supprimerDonnees(id) {
  await client.delete(id);
  console.log('Données supprimées avec succès');
}
```

## Partie 3 : Gestion des erreurs

### Capturer les erreurs

```javascript
async function gererErreurs() {
  try {
    const result = await client.get('/donnees/inexistant');
  } catch (error) {
    if (error.code === 'NOT_FOUND') {
      console.log('Ressource non trouvée');
    } else if (error.code === 'UNAUTHORIZED') {
      console.log('Accès non autorisé');
    } else {
      console.error('Erreur inattendue:', error);
    }
  }
}
```

### Utiliser les retry automatiques

```javascript
const clientAvecRetry = new Client({
  apiKey: process.env.API_KEY,
  retry: {
    maxAttempts: 3,
    backoff: 'exponential'
  }
});
```

## Partie 4 : Patterns avancés

### Utiliser les callbacks

```javascript
client.on('request', (config) => {
  console.log('Requête:', config.method, config.url);
});

client.on('response', (response) => {
  console.log('Réponse:', response.status);
});

client.on('error', (error) => {
  console.error('Erreur globale:', error);
});
```

### Chaîner les opérations

```javascript
async function operationsChainées() {
  const result = await client
    .create({ nom: 'Test' })
    .then(donnee => client.update(donnee.id, { status: 'actif' }))
    .then(donnee => client.addTags(donnee.id, ['important']))
    .then(donnee => {
      console.log('Opérations terminées:', donnee);
      return donnee;
    });

  return result;
}
```

## Exercices pratiques

### Exercice 1 : CRUD complet

Créez un script qui :
1. Crée une nouvelle donnée
2. La lit pour vérifier
3. La met à jour
4. La supprime

### Exercice 2 : Gestion d'erreurs

Implémentez un système de retry manuel avec un délai exponentiel.

### Exercice 3 : Pipeline de données

Créez un pipeline qui traite des données en série avec transformation.

## Résumé

Dans ce tutoriel, vous avez appris :

- ✅ Créer et configurer un client
- ✅ Effectuer des opérations CRUD
- ✅ Gérer les erreurs efficacement
- ✅ Utiliser des patterns avancés

## Prochaines étapes

- Continuez avec le [tutoriel avancé](advanced-tutorial.md)
- Explorez les [cas d'usage](use-cases.md)
- Consultez la [référence API complète](../api-reference/introduction.md)
