# Distributed Redis Cache Setup Kind

Here’s a **step-by-step guide** to build a **distributed Redis cache** using **Kubernetes in Docker (Kind)** and use that Redis cache in a **Spring Boot application**.

---

## 🔧 Prerequisites

* Docker
* Kind (Kubernetes in Docker) installed
* kubectl installed
* Helm (for Redis chart deployment)
* Java + Maven or Gradle
* Spring Boot starter project

---

## ✅ Step 1: Create a Kind Cluster

```bash
brew install kind 

kind create cluster --name redis-cluster
```

You can customize the Kind config if you need specific port mappings.

---

## ✅ Step 2: Add Helm Repo & Install Redis Cluster

Add the Bitnami Helm chart repository:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Install a Redis cluster using Helm:

```bash
helm install redis-cluster bitnami/redis-cluster \
  --set cluster.enabled=true \
  --set cluster.slaveCount=1 \
  --set auth.enabled=false \
  --set global.storageClass=standard
```

> ⚠️ You can enable authentication and persistence in production. For dev/demo, we disable them for simplicity.

Wait for pods to be ready:

```bash
kubectl get pods -w
```

---

## ✅ Step 3: Port Forward Redis to Local Machine

```bash
kubectl port-forward svc/redis-cluster 6379:6379
```

This allows your Spring Boot app to connect to Redis via `localhost:6379`.

---

## ✅ Step 4: Configure Spring Boot for Redis

Add dependencies in `pom.xml` or `build.gradle`:

**Maven:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**Gradle:**

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

---

## ✅ Step 5: Spring Boot Application Properties

Add to `application.properties` or `application.yml`:

```properties
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379
```

---

## ✅ Step 6: Enable Caching in Your App

```java
@SpringBootApplication
@EnableCaching
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

Add caching to your service methods:

```java
@Service
public class MyService {
    @Cacheable("items")
    public String getItem(String id) {
        return expensiveOperation(id);
    }
}
```

---

## ✅ Step 7: Test the Setup

* Start your Spring Boot app.
* Hit the endpoint that triggers the `@Cacheable` method.
* Observe that the second call is much faster and served from Redis.

---

## 🧪 Optional: Check Redis Content (CLI)

You can exec into the Redis pod:

```bash
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=redis-cluster -o jsonpath="{.items[0].metadata.name}") -- bash
```

And run Redis CLI:

```bash
redis-cli
```

Then run:

```bash
KEYS *
GET your_cache_key
```

---

## 🧹 Cleanup

```bash
kind delete cluster --name redis-cluster
```

---

Would you like a sample Spring Boot project with Redis caching setup to test with this?
