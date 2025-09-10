# Keycloak-gke-deploy
This repository contains deployment code for keycloak on GKE.

# Prerequisites
- kubectl CLI

# Deployment without cert-manager
1. apply yaml files sequentially
   - namespace.yml 
   - gar-secrets.yml 
   - keycloak-secrets.yml 
   - keycloak.yml 
   - ingress.yml
2. TLS certifcates
   - generate CA authority signed certs for your custom domain
     - ref: https://medium.com/sslforweb/generate-free-lets-encrypt-ssl-certificate-e18e9db6ebac
   - provide PEM certificates as a kubernetes secret 
     - kubectl create secret tls <my-tls-secret-name> --cert=cert.pem --key=key.pem -n kc-azure-prod
   - map tls secret in ingress under: 
     - spec.tls.hosts.secretname


# Deployment with cert-manager
1. apply yaml files sequentially
   - <kubectl-apply-yaml-file-command> namespace.yml 
   - <..> gar-secrets.yml 
   - <..> keycloak-secrets.yml 
   - <..> keycloak.yml 
2. first apply ingress with http only (i.e without tls mention in ingress & without adding cert-manager annotation)
   - <..> ingress.yml
<!-- note: wait for host name to be accessible on http (typically takes 5-10 mins) -->
2. TLS certifcates
   - apply cert-manager & issuer 
     - <kubectl-apply-yaml-file-command> issuer-prod.yml
   - create k8s certificate secret
     - <..> tls-secrets-cert-mngr.yml
   - map tls secret in ingress under: 
     - spec.tls.hosts.secretname
   - add annotations in ingress 
     - cert-manager.io/issuer: letsencrypt-production
     - acme.cert-manager.io/http01-edit-in-place: "true"


# For image build locally and push to google artifact registry
1. base64 encoding of service account key 
   - cat key.json > key-base64.json
2. docker login manually on machine
   - cat key-base64.json | docker login -u _json_key_base64 --password-stdin <docker registry url>
3. custom docker image build locally 
   - docker build -t <locally-builded-image-name> . --no-cache
4. tag docker image according to gcp project id & artifact registry name & image name on artifact
   - docker tag <locally-builded-image-name> <asia-south1-docker.pkg.dev>/<PROJECT-ID>/<REPOSITORY>/<IMAGE-NAME-ON-ARTIFACT>:TAG 
5. push image to artifact
   - docker push <asia-south1-docker.pkg.dev>/<PROJECT-ID>/<REPOSITORY>/<IMAGE-NAME-ON-ARTIFACT>:TAG
ref: 
- https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling#tag
- https://stackoverflow.com/questions/78000747/how-to-pull-docker-image-from-google-artifact-registry-in-k8s-deployment-yaml-vi


# IMP commands 
1. set project in GCP 
   - gcloud config set project <PROJECT-ID>
2. create static IP in gcp project 
   - gcloud compute addresses create <static-ip-name> --global
3. List all pods in namespaces
   - kubectl get pods -n <namespace-name>
4. creating cluster command with VPC network specified 
   gcloud container clusters create <cluster-name> --project=<project-name> --region=<region> \
   --network=<network-name> --subnetwork=<sub-network-name> --num-nodes=<num-of-nodes> --machine-type=e2-medium



# Reference key examples 
1. <docker registry url> for asia-south1 : https://asia-south1-docker.pkg.dev
2. <TAG> is optional while pushing & pulling images
