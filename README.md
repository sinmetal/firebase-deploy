# firebase-deploy

事前準備

Workload Identityを使う設定は https://zenn.dev/vvakame/articles/gha-and-gcp-workload-identity の通り
この設定にFirebaseを使う権限設定を追加する

```
 export PROJECT_ID=firebase-deploy-20230110
 export POOL_NAME=github-actions
 export PROVIDER_NAME=gha-provider
 export SA_EMAIL=github-actions@firebase-deploy-20230110.iam.gserviceaccount.com
 export GITHUB_REPO=sinmetal/firebase-deploy

 gcloud services enable iamcredentials.googleapis.com --project "${PROJECT_ID}"
 gcloud services enable firebasehosting.googleapis.com --project "${PROJECT_ID}"
 gcloud iam workload-identity-pools create "${POOL_NAME}" --project="${PROJECT_ID}" --location="global" --display-name="use from GitHub Actions"
 export WORKLOAD_IDENTITY_POOL_ID=$( \\n    gcloud iam workload-identity-pools describe "${POOL_NAME}" \\n      --project="${PROJECT_ID}" --location="global" \\n      --format="value(name)" \\n  )
 echo $WORKLOAD_IDENTITY_POOL_ID
 gcloud iam workload-identity-pools providers create-oidc "${PROVIDER_NAME}" --project="${PROJECT_ID}" --location="global" --workload-identity-pool="${POOL_NAME}" --display-name="use from GitHub Actions provider" --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository,attribute.actor=assertion.actor,attribute.aud=assertion.aud" --issuer-uri="https://token.actions.githubusercontent.com"
 gcloud iam service-accounts add-iam-policy-binding "${SA_EMAIL}" --project="${PROJECT_ID}" --role="roles/iam.workloadIdentityUser" --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${GITHUB_REPO}"

 gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_EMAIL}" --role="roles/firebase.developAdmin"
 ```