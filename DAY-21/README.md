
# Kubernetes Certificate Signing Request (CSR) Creation and Approval

This guide will walk you through the steps to generate a private key, a Certificate Signing Request (CSR), create a Kubernetes CertificateSigningRequest, approve it, retrieve the issued certificate, and export it.

## Steps

### 1. Generate Private Key and CSR

First, generate a private key and CSR using OpenSSL.

```bash
# Generate the private key
openssl genrsa -out learner.key 2048

# Generate the CSR with Common Name (CN) as 'learner'
openssl req -new -key learner.key -out learner.csr -subj "/CN=learner"
```

### 2. Base64 Encode the CSR

To create a Kubernetes CSR object, you need to base64 encode the CSR.

```bash
# Encode the CSR to base64
cat learner.csr | base64 | tr -d '\n'
```

### 3. Create Kubernetes CertificateSigningRequest (CSR)

Create a YAML file named `learner-csr.yaml` with the base64-encoded CSR content in the `request` field.

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: learner
spec:
  request: <BASE64_ENCODED_CSR>  # Replace this with the actual base64 encoded CSR
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
  expirationSeconds: 604800  # 1 week (604800 seconds)
```

Apply the CSR:

```bash
kubectl apply -f learner-csr.yaml
```

### 4. Approve the CSR

Approve the CertificateSigningRequest (CSR) using the following command:

```bash
kubectl certificate approve learner
```

### 5. Retrieve the Issued Certificate

Once the CSR is approved, retrieve the issued certificate and decode it:

```bash
# Decode and save the issued certificate to learner.crt
kubectl get csr learner -o jsonpath='{.status.certificate}' | base64 --decode > learner.crt
```

### 6. Export the Issued Certificate in YAML

If you want to export the full CertificateSigningRequest resource, including the issued certificate, to a YAML file:

```bash
# Export the CSR with the certificate
kubectl get csr learner -o yaml > learner-csr.yaml
```

### Summary

- **Private Key**: `learner.key`
- **Certificate Signing Request**: `learner.csr`
- **Issued Certificate**: `learner.crt`

This process creates, approves, and retrieves a signed certificate from Kubernetes, with an expiration of one week.

