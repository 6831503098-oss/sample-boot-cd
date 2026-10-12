# Sample: Continuous Deployment — Docker + Cloud Build + Cloud Run

In-class demo for week 10: take a finished Spring Boot app (customer API +
H2) and make every `git push` build a Docker image and deploy it to Google
Cloud Run automatically.

**➡️ Follow the demonstration flow in [TODO.md](TODO.md).**

- Branch `main` — in-class version: the app is complete, the DEPLOYMENT
  pieces are what we build together (Dockerfile live, project id in
  `cloudbuild.yaml`)
- Branch [`complete`](https://github.com/cnacha-mfu/sample-boot-cd/tree/complete) —
  finished Dockerfile + cloudbuild.yaml for reference
- Prepares you for the graded lab:
  [lab-web-cd](https://github.com/maefahluang-uni/lab-web-cd)

Run the app locally:

```
mvn spring-boot:run        # http://localhost:8080/customers
```

##