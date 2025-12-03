## Common Errors & Fixes

### 1. Ingress webhook error
kubectl delete validatingwebhookconfiguration ingress-nginx-admission

### 2. GatewayClass immutable error
Controller created it already → Do NOT create again.

### 3. No Service for nginx-gateway
Create nginx-gateway-service.yaml

