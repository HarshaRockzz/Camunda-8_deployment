# Challengers and Resolving Common AWS Issues


## 1. IAM Permission Errors

**Problem:**
Encountered permission errors while attempting to execute AWS operations.

**Solution:**
- Attached appropriate IAM policies, such as:
  - `AdministratorAccess` for full access.
  - Specific ECS/EKS permissions based on the required operations.


## 2. ECS Task Failures

**Problem:**
ECS tasks were failing during execution.

**Solution:**
- Addressed resource allocation issues in ECS task definitions (e.g., CPU and memory settings).
- Used AWS CloudWatch logs to debug and identify specific errors.

## 3. Load Balancer Issues

**Problem:**
Load balancer was not routing traffic correctly.

**Solution:**
- Ensured that the VPC had an Internet Gateway attached.
- Configured security groups to allow inbound traffic on required ports (e.g., HTTP: 80, HTTPS: 443).

## 4. Helm Chart Configuration Errors

**Problem:**
Deployment using Helm charts failed due to incorrect configurations.

**Solution:**
- Created a custom `values.yaml` file with the correct configurations for:
  - PostgreSQL settings (e.g., database name, username, password).
  - Zeebe settings as required for the application.

---

