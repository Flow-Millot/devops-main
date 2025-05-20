# TP part 02

### 2-1 What are testcontainers?

Testcontainers are a Java library that helps us to run lightweight, throwaway Docker containers directly from our automated tests. They make it easy to test our code.

### 2-2 For what purpose do we need to use secured variables ?

Secured variables protect sensitive information like passwords, tokens, and keys from being exposed in code or logs when we push it. They prevent leaks, restrict access to authorized users only and workflows only. Theykeep our accounts and services safe from unauthorized access.

### 

We use `needs: build-and-test-backend` to be sure that this job runs only after the backend tests pass. This prevents building and pushing Docker images if tests fail, because we will save time and ressources and ensure that only verified code is published.

