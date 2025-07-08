
# Secret Management Documentation

## Overview

This document outlines the secret management strategy for the Finch project in Kubernetes. It explains how sensitive information such as database credentials and API keys are securely stored, managed, and accessed by the applications.

---

## Single Kubernetes Secret File for All Sensitive Data

For simplicity and ease of management, all sensitive environment variables and secrets are stored in a **single Kubernetes Secret manifest**:

- The secret named `django-env-secret` contains database credentials, API keys, and other environment variables required by the backend.
- This secret is applied in the `capstone` namespace.
- In deployment manifests, this secret is referenced using `envFrom.secretRef` to inject **all** key-value pairs as environment variables inside the container.
- This reduces complexity by managing one secret resource instead of multiple.
- **Important:** Access to this secret should be tightly controlled using Kubernetes RBAC policies to prevent unauthorized exposure.

---

## Example Secret Manifest

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: django-env-secret
  namespace: capstone
type: Opaque
stringData:
  POSTGRES_DB: fleetdb
  POSTGRES_USER: roy77
  POSTGRES_PASSWORD: asdf1234@77
  POSTGRES_HOST: postgres
  POSTGRES_PORT: "5432"
  PYTHONUNBUFFERED: "1"
  STRIPE_SECRET_KEY: "sk_test_51R2EHqFxSoY5Rjyor0IeGElOkgFZMziWh7YMbEydw7Xrby1GlzZijsW3JXw7sqhTRWTKKTIsG4xxPXhlCEQd5dRU00zvsQANcy"
  STRIPE_PUBLIC_KEY: "pk_test_51R2EHqFxSoY5RjyoISdnIQZkHvt6LmxJdpdEVxG3MYvlhbkgOQPzTUck2pZtcbKLQQKasG7VHZNkzMH7bCv6XmC600QlbooiKJ"
  STRIPE_WEBHOOK_SECRET: "whsec_uAoNngWqlE3qITAux0SEVXMcMdv5yT1I"
  SITE_URL: "http://93.127.195.189:5000"
  FRONTEND_SITE_URL: "https://finch-development.vercel.app"
```

---

## How to Apply and Update the Secret

### Apply the Secret Manifest

To create or update the secret in your Kubernetes cluster, run:

```bash
kubectl apply -f kubernetes/secrets/dep-secrets.yml
```

This will create the secret if it doesn't exist or update it if it does.

### Update Specific Secret Values Manually

If you want to update specific secret values without editing the YAML file directly, you can use `kubectl create secret generic` with `--dry-run=client` and apply it:

```bash
kubectl create secret generic django-env-secret \
  --namespace=capstone \
  --from-literal=POSTGRES_PASSWORD=new_secure_password \
  --dry-run=client -o yaml | kubectl apply -f -
```

---

## How to Reference the Secret in Your Deployment

In your backend deployment manifest, use the following to inject all secrets as environment variables:

```yaml
envFrom:
  - secretRef:
      name: django-env-secret
```

This makes all keys in the secret available as environment variables in the container, for example:

- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PASSWORD`
- `STRIPE_SECRET_KEY`
- and others...

---

## Security Best Practices

- **Avoid storing secrets in plain text in public repositories.** Keep secret manifests in private or encrypted storage.
- **Limit access to secrets using Kubernetes Role-Based Access Control (RBAC).** Only authorized service accounts and users should be able to read secrets.
- **Rotate secrets regularly** to reduce the risk of exposure.
- **Consider integrating with external secret managers** such as HashiCorp Vault, AWS Secrets Manager, or Azure Key Vault for enhanced security in production environments.
- **Do not log secret values** anywhere in your application or CI/CD pipelines.

---

## Summary

By using a single Kubernetes Secret manifest, Finch achieves simplified secret management, ensuring all sensitive information is securely stored and seamlessly injected into application pods while maintaining good security hygiene.
