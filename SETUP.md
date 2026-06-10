# Guide de connexion GitHub + Jenkins + Snyk

## Structure du repo GitHub

```
web-app/
├── app.js
├── app.test.js
├── package.json
├── Dockerfile
├── .gitignore
└── Jenkinsfile
```

---

## 1. Créer le repo GitHub et pousser le code

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/TON_USER/web-app.git
git push -u origin main
```

---

## 2. Connecter Jenkins à GitHub

### a) Créer un Personal Access Token GitHub
1. GitHub → Settings → Developer settings → Personal access tokens → **Tokens (classic)**
2. Scopes requis : `repo`, `admin:repo_hook`
3. Copie le token généré

### b) Ajouter le credential dans Jenkins
1. Jenkins → Manage Jenkins → Credentials → System → Global credentials
2. **Add Credentials** :
   - Kind : `Username with password`
   - Username : ton pseudo GitHub
   - Password : le token GitHub
   - ID : `github-token`  ← doit correspondre au `credentialsId` dans le Jenkinsfile

### c) Créer le pipeline Jenkins
1. New Item → **Pipeline**
2. Dans "Pipeline" → Definition : **Pipeline script from SCM**
3. SCM : **Git**
4. Repository URL : `https://github.com/TON_USER/web-app.git`
5. Credentials : sélectionner `github-token`
6. Branch : `*/main`
7. Script Path : `Jenkinsfile`

### d) Configurer le webhook GitHub (déclenchement automatique)
1. Dans Jenkins : Manage Jenkins → Configure System → cocher **GitHub Hook Trigger**
2. Dans GitHub : repo → Settings → Webhooks → **Add webhook**
   - Payload URL : `http://TON_JENKINS:8080/github-webhook/`
   - Content type : `application/json`
   - Trigger : `Just the push event`
3. Dans ton pipeline Jenkins : cocher **GitHub hook trigger for GITScm polling**

---

## 3. Connecter Snyk

### a) Récupérer ton token Snyk
1. Connecte-toi sur https://app.snyk.io
2. Account Settings → **Auth Token** → copie le token

### b) Ajouter le token dans Jenkins
1. Jenkins → Manage Jenkins → Credentials → System → Global credentials
2. **Add Credentials** :
   - Kind : `Secret text`
   - Secret : ton token Snyk
   - ID : `snyk-token`  ← doit correspondre à `credentials('snyk-token')` dans le Jenkinsfile

### c) Installer Snyk CLI sur l'agent Jenkins
```bash
npm install -g snyk
# ou via npm sans install global :
npx snyk --version
```

---

## 4. Résumé des credentials Jenkins requis

| ID             | Type                  | Contenu              |
|----------------|-----------------------|----------------------|
| `github-token` | Username with password | Login + PAT GitHub  |
| `snyk-token`   | Secret text           | Token Snyk           |

---

## 5. Ce que fait le pipeline

| Étape | Action |
|-------|--------|
| 01 | Affiche les paramètres |
| 02 | Clone le repo GitHub sur la branche choisie |
| 03 | `npm install` |
| 04 | `snyk test` — scan des vulnérabilités npm (échoue sur HIGH/CRITICAL) |
| 05 | `npm test` — tests Jest (ignoré si SKIP_TEST=true) |
| 06 | Analyse SonarQube |
| 07 | `docker build` |
| 08 | `docker run` sur le port 3000 |
