# Spring Boot CI/CD: GitHub -> Docker -> Kubernetes

## Flow
git push -> test -> build image -> validate -> push to GHCR -> deploy -> verify

| Stage | What it does | Stops the pipeline if |
|---|---|---|
| Test | `mvn verify` | code does not compile or a test fails |
| Build | multi-stage Dockerfile | image build fails |
| Validate | health check, non-root check, Trivy scan | app unhealthy, root user, fixable CRITICAL CVE |
| Push | image tagged with the commit SHA | registry error |
| Deploy | applies `k8s/` to a kind cluster | rollout or HTTP check fails |

Pull requests run test, build and validate only.

## How to run it
1. Create an empty GitHub repository.
2. In this folder:

       git init
       git add .
       git commit -m "Add Spring Boot CI/CD pipeline"
       git branch -M main
       git remote add origin https://github.com/<your-username>/<repo>.git
       git push -u origin main

3. Open the repository's Actions tab and watch the CI-CD run.
4. To prove it is automatic, change the message in DemoApplication.java, commit and push.

No secrets need to be added; the built-in GITHUB_TOKEN is used.

## Using your organization's application
Copy Dockerfile, .dockerignore, k8s/ and .github/ into its repository, then:
- add spring-boot-starter-actuator and the properties in application.properties
- set `<finalName>app</finalName>` in pom.xml (or edit the COPY line in the Dockerfile)
- match the Java version in the Dockerfile and workflow
- change the port if it is not 8080
- for Gradle, replace the Maven commands
- provide any database or config the app needs to start (otherwise the health check fails)

## Persistent cluster
The kind cluster exists only while the job runs. It demonstrates and tests deployment.
For a lasting environment (Minikube on your own machine, or a cloud cluster):
- install a self-hosted GitHub runner on a machine with kubectl access (use a private repository)
- in the deploy job set `runs-on: self-hosted` and remove the cluster creation and image load steps
- make the GHCR package public, or add an imagePullSecret
- roll back with `kubectl rollout undo deployment/demo-app`
