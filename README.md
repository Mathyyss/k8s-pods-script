https://github.com/kubernetes-client/javascript/blob/main/examples/example.js
Très bien, tu veux maintenant faire une interface UI mais au lieu d’utiliser directement listPodForAllNamespaces(),
tu veux exécuter des vraies commandes Kubernetes (kubectl get pods) depuis Node.js via exec.

C’est une approche différente (et totalement possible) !

Voici l’approche claire étape par étape que je vais te faire :

Backend :
	•	Utiliser child_process.exec pour exécuter kubectl get pods -A -o json
	•	Récupérer la sortie JSON
	•	L’envoyer au frontend

Frontend :
	•	Afficher les pods dans un tableau HTML (pareil que tout à l’heure)

Étapes détaillées

1. Ton environnement (pareil)
	•	Node.js installé
	•	Kubernetes cluster qui tourne (kubectl OK)

Teste :

kubectl get pods -A -o json

Ça doit te donner un gros JSON.

2. Préparer ton projet

Si tu n’as pas encore de projet :

mkdir k8s-pods-ui
cd k8s-pods-ui
npm init -y
npm install express

3. Le Backend (server.js)

Crée server.js :

const express = require('express');
const { exec } = require('child_process');
const path = require('path');

const app = express();
const port = 5000;

// Route pour récupérer les pods
app.get('/api/pods', (req, res) => {
    exec('kubectl get pods -A -o json', (error, stdout, stderr) => {
        if (error) {
            console.error('Erreur exec:', error);
            return res.status(500).json({ error: 'Erreur récupération pods' });
        }
        try {
            const pods = JSON.parse(stdout);
            res.json(pods);
        } catch (err) {
            console.error('Erreur parse JSON:', err);
            res.status(500).json({ error: 'Erreur parsing JSON' });
        }
    });
});

// Sert le frontend
app.use(express.static('frontend'));

app.listen(port, () => {
    console.log(`Server running on http://localhost:${port}`);
});

Explication rapide :
	•	Quand tu appelles /api/pods, il exécute kubectl get pods -A -o json.
	•	Il lit la sortie et te la renvoie en JSON.

4. Le Frontend (frontend/index.html)

Crée frontend/index.html :

<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Liste des Pods Kubernetes</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 30px;
        }
        table {
            border-collapse: collapse;
            width: 100%;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
        }
        th {
            background-color: #f4f4f4;
        }
    </style>
</head>
<body>

<h1>Liste des Pods (via kubectl exec)</h1>

<table id="podsTable">
    <thead>
        <tr>
            <th>Nom</th>
            <th>Namespace</th>
            <th>Statut</th>
            <th>Date de création</th>
        </tr>
    </thead>
    <tbody>
    </tbody>
</table>

<script>
async function fetchPods() {
    try {
        const response = await fetch('/api/pods');
        const data = await response.json();
        const tableBody = document.querySelector('#podsTable tbody');
        tableBody.innerHTML = '';

        if (data.items && Array.isArray(data.items)) {
            data.items.forEach(pod => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${pod.metadata.name}</td>
                    <td>${pod.metadata.namespace}</td>
                    <td>${pod.status.phase}</td>
                    <td>${new Date(pod.metadata.creationTimestamp).toLocaleString()}</td>
                `;
                tableBody.appendChild(row);
            });
        } else {
            tableBody.innerHTML = '<tr><td colspan="4">Aucun pod trouvé.</td></tr>';
        }
    } catch (error) {
        console.error('Erreur lors de la récupération des pods:', error);
    }
}

// Charger au démarrage
fetchPods();

// Auto-refresh (optionnel)
setInterval(fetchPods, 10000);
</script>

</body>
</html>

5. Lancer ton serveur

node server.js

6. Ouvrir ton navigateur

Va sur :

http://localhost:5000

Et tu verras ton tableau de pods Kubernetes !

Pourquoi c’est pratique d’utiliser exec ?
	•	Pas besoin d’utiliser la lib Kubernetes officielle.
	•	Simple, tu fais juste confiance à kubectl.
	•	Rapide à coder.

Mais attention : ça dépend de kubectl présent et du bon contexte.

Résumé de cette méthode

Action	Détail
Backend	Node.js + child_process.exec("kubectl get pods -A -o json")
Frontend	HTML simple qui consomme /api/pods
Rafraîchissement	Auto toutes les 10 secondes (facultatif)

Veux-tu maintenant que je te fasse directement une version avec un meilleur design Bootstrap ?
Ou ajouter : une colonne “IP du pod”, “Restart count”, etc. ?

Je peux aussi te montrer comment ajouter un bouton “Rafraîchir” !
Veux-tu que je continue ?
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Liste des Pods Kubernetes</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 30px;
        }
        table {
            border-collapse: collapse;
            width: 100%;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
        }
        th {
            background-color: #f4f4f4;
        }
    </style>
</head>
<body>

<h1>Liste des Pods (via kubectl exec)</h1>

<table id="podsTable">
    <thead>
        <tr>
            <th>Nom</th>
            <th>Namespace</th>
            <th>Statut</th>
            <th>Date de création</th>
        </tr>
    </thead>
    <tbody>
    </tbody>
</table>

<script>
async function fetchPods() {
    try {
        const response = await fetch('/api/pods');
        const data = await response.json();
        const tableBody = document.querySelector('#podsTable tbody');
        tableBody.innerHTML = '';

        if (data.items && Array.isArray(data.items)) {
            data.items.forEach(pod => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${pod.metadata.name}</td>
                    <td>${pod.metadata.namespace}</td>
                    <td>${pod.status.phase}</td>
                    <td>${new Date(pod.metadata.creationTimestamp).toLocaleString()}</td>
                `;
                tableBody.appendChild(row);
            });
        } else {
            tableBody.innerHTML = '<tr><td colspan="4">Aucun pod trouvé.</td></tr>';
        }
    } catch (error) {
        console.error('Erreur lors de la récupération des pods:', error);
    }
}

// Charger au démarrage
fetchPods();

// Auto-refresh (optionnel)
setInterval(fetchPods, 10000);
</script>

</body>
</html>
