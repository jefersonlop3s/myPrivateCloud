# Notes

## Cert-Manager Install

  ```
  kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.crds.yaml
  helm repo add jetstack https://charts.jetstack.io
  helm update
  helm install cert-manager jetstack/cert-manager   --namespace cert-manager   --create-namespace   --version v1.13.1
  
  kubectl apply -f serlfSignedCertManager.yaml
  ```