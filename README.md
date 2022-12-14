# Deploy Firebase function with environnement target

Deploy Firebase functions with environments for better continuous integration. Need to deploy to production on one branch, then to development on another? Specify a 'target' by using the <key, value> set in your `firebaserc` file. For example:

```
{
  "projects": {
    "default": "default project id",
    "production": "production project id",
    "staging": "staging project id"
  }
}
```

This action use [https://github.com/marketplace/actions/authenticate-to-google-cloud](https://github.com/marketplace/actions/authenticate-to-google-cloud) for Google services authentication

## Arguments

---

Firebase env name, see [https://firebase.google.com/docs/projects/dev-workflows/overview-environments](https://firebase.google.com/docs/projects/dev-workflows/overview-environments)

```
TARGET: 'default' or your env name
```

Add this if you want to deploy storage rules

```
DEPLOY_STORAGE: true
```

Add this if you want to deploy firestore rules

```
DEPLOY_FIRESTORE_RULES: true
```

Add this if you want to deploy firestore index rules

```
DEPLOY_FIRESTORE_INDEX: true
```

Add this if you want to deploy database rules

```
DEPLOY_FIRESTORE_INDEX: true
```

## Instructions

1. Get your Firebase service account json from firebase admin for each of your project env

2. Create a GOOGLE_CREDENTIALS_STAGING and GOOGLE_CREDENTIALS_PRODUCTION in you Github repo secret, add your firebase credentials. If you don't have multiples firebase env create GOOGLE_CREDENTIALS secret only and past corresponding Firebase service account json.

### Example production workflow

---

```
name: Deploy production

"on":
  release:
    types: [published]

jobs:
  deploy_hosting:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci && npm run build
      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: "${{ secrets.GITHUB_TOKEN }}"
          firebaseServiceAccount: "${{ secrets.FIREBASE_SERVICE_ACCOUNT_PRODUCTION }}"
          channelId: live
          projectId: your-project-id

  deploy_functions:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - id: "auth"
        name: "Authenticate to Google Cloud"
        uses: "google-github-actions/auth@v1"
        with:
          credentials_json: "${{ secrets.GOOGLE_CREDENTIALS_STAGING_PRODUCTION }}"
      - name: Firebase CI Ops
        uses: flame-app-studio/firebase-ci-ops@v1.3.0
        env:
          TARGET: production
          DEPLOY_STORAGE: true
          DEPLOY_FIRESTORE_RULES: true
          DEPLOY_FIRESTORE_INDEX: true
```

### Example staging workflow

---

```
name: Deploy staging

"on":
  push:
    branches:
      - develop

jobs:
  deploy_hosting:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci && npm run build
      - uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: "${{ secrets.GITHUB_TOKEN }}"
          firebaseServiceAccount: "${{ secrets.FIREBASE_SERVICE_ACCOUNT_STAGING }}"
          channelId: live
          projectId: your-project-id

  deploy_functions:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - id: "auth"
        name: "Authenticate to Google Cloud"
        uses: "google-github-actions/auth@v1"
        with:
          credentials_json: "${{ secrets.GOOGLE_CREDENTIALS_STAGING }}"
      - name: Firebase CI Ops
        uses: flame-app-studio/firebase-ci-ops@v1.3.0
        env:
          TARGET: staging
          DEPLOY_STORAGE: true
          DEPLOY_FIRESTORE_RULES: true
          DEPLOY_FIRESTORE_INDEX: true
```

<br/><br/>

**_Crafted with ?????? by Jean-Baptiste Thery | Flame App Studio | [www.flameapp.studio](www.flameapp.studio)_**
