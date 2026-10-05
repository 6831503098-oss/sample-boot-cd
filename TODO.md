# In-Class Demonstration Flow — Continuous Deployment

This repository has two branches:

- **`main`** — the in-class version. The Spring Boot app is COMPLETE; what we
  build together is the deployment: the `Dockerfile` (does not exist yet) and
  your project id in `cloudbuild.yaml`.
- **[`complete`](https://github.com/cnacha-mfu/sample-boot-cd/tree/complete)** —
  the finished `Dockerfile` and `cloudbuild.yaml` to catch up or check against.

The goal of the hour: **a `git push` that deploys itself.**

---

## Step 1: The app we are deploying

**Idea:** CD starts with a working app. Ours is the customer API from the
fullstack week — nothing new inside it.

**Try it:** `mvn spring-boot:run`, open <http://localhost:8080/customers> —
20 customers as JSON. Stop it with Ctrl+C.

---

## Step 2: Docker — build and run the app as a container

**File:** `Dockerfile` (we create it now, in the project root)

**Idea:** a container packages the app AND everything it needs. The image is
built in two stages: a big image with Maven builds the jar, a small image
with only Java runs it.

**Fill in together** (type it live):

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS build
COPY src /home/app/src
COPY pom.xml /home/app
RUN mvn -f /home/app/pom.xml clean package

# Package stage
FROM eclipse-temurin:17-jre
COPY --from=build /home/app/target/*.jar /usr/local/lib/app.jar
ENTRYPOINT ["java","-jar","/usr/local/lib/app.jar"]
```

**Try it** (the docker commands from the lecture — build, run):

```
docker build -t sample-cd .
docker run --rm -p 8080:8080 sample-cd
```

<http://localhost:8080/customers> again — same app, now inside a container.

**Solution:** [Dockerfile](https://github.com/cnacha-mfu/sample-boot-cd/blob/complete/Dockerfile)

---

## Step 3: cloudbuild.yaml — the pipeline as code

**File:** `cloudbuild.yaml` (already in the repo — read it together)

**Idea:** three steps, the same three verbs from the lecture:
**build** the image → **push** it to Artifact Registry → **deploy** it to
Cloud Run. The pipeline lives IN the repository, next to the code.

**Fill in together:** put your own GCP project id into
`substitutions._PROJECT_ID`.

**Solution:** [cloudbuild.yaml](https://github.com/cnacha-mfu/sample-boot-cd/blob/complete/cloudbuild.yaml)

---

## Step 4: One-time setup in the Google Cloud console

**Idea:** Cloud Build needs a place to store images, permission to deploy,
and a trigger that watches the repository.

Do together at <https://console.cloud.google.com>:

1. **Artifact Registry** → create repository → name it `webdev-dockers`
   (region `us-central1`, format Docker).
2. **Cloud Build → Repositories** → Connect repository → GitHub →
   select this repo.
3. **Cloud Build → Triggers** → Create trigger → name `sample-cd`,
   event *Push to a branch*, source this repo, configuration
   *Cloud Build configuration file* (`cloudbuild.yaml`).
4. **Cloud Build → Settings** → enable **Cloud Run Admin** for the Cloud
   Build service account.

---

## Step 5: Push — and watch it deploy itself

**Try it:** commit the Dockerfile and the project id, `git push`. Then watch,
in order:

1. **Cloud Build → History** — a build appears, running our three steps.
2. **Cloud Run** — a `sample-cd` service appears when the build goes green.
3. Open the service URL, add `/customers` — the SAME app, now on the
   public internet.

---

## Step 6: The point of CD — change, push, redeployed

**Try it:** change something visible (edit a customer in `data.sql`, or a
field name in the JSON), commit, push. Reload the Cloud Run URL a minute
later — the change is live. Nobody touched a server.

**Idea to say out loud:** this is the DevOps loop from the lecture — one
repository carrying the functional code AND the environment (Dockerfile) AND
the pipeline (cloudbuild.yaml).

---

## After class

The graded lab, [lab-web-cd](https://github.com/maefahluang-uni/lab-web-cd),
is this exact flow on the concert app, with two differences to notice:

| In the demo | In the lab |
|---|---|
| App listens on 8080 (Cloud Run's default) | App listens on **8100** — the deploy step needs `--port 8100` |
| We checked the URL by eye | You paste your service URL into `SERVICE_URL.txt` — the autograder calls your LIVE service and marks you |

> **Tip:** Docker commands need your LOCAL machine (Docker Desktop) — a
> Codespace cannot run Docker Desktop, and the Cloud Build path works from
> any machine because the building happens in the cloud.
