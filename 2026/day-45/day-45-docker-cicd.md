### Task 1: Prepare
1. Use the app you Dockerized on Day 36 (or any simple Dockerfile)
```
$ vim app.py

from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Dockerized Python App 🚀"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
```
$ vim requirements.txt

flask
```
2. Add the Dockerfile to your `github-actions-practice` repo (or create a minimal one)
```
$ vim Dockerfile

FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```
3. Make sure `DOCKER_USERNAME` and `DOCKER_TOKEN` secrets are set from Day 44
<img width="1358" height="344" alt="image" src="https://github.com/user-attachments/assets/60a6321e-39e1-4af9-9ce7-933c6e3d4950" />

---

### Task 2: Build the Docker Image in CI
Create `.github/workflows/docker-publish.yml` that:
1. Triggers on push to `main`
2. Checks out the code
3. Builds the Docker image and tags it
```
$ vim docker-build.yml

name: Build Docker Image

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build -t my-docker-app:latest .
```
**Verify:** Check the build step logs — does the image build successfully?
<img width="1831" height="669" alt="image" src="https://github.com/user-attachments/assets/2eea836c-e58e-4573-b7b5-16405427dc1f" />

---

### Task 3: Push to Docker Hub
Add steps to:
1. Log in to Docker Hub using your secrets
2. Tag the image as `username/repo:latest` and also `username/repo:sha-<short-commit-hash>`
3. Push both tags
```
name: Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up short SHA
        id: vars
        run: echo "SHA_SHORT=$(echo $GITHUB_SHA | cut -c1-7)" >> $GITHUB_ENV

      - name: Log in to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_TOKEN }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/my-app:latest .

      # use existing image and tag the image with sha
      - name: Tag image with commit SHA
        run: |
          docker tag ${{ secrets.DOCKER_USERNAME }}/my-app:latest \
          ${{ secrets.DOCKER_USERNAME }}/my-app:sha-${{ env.SHA_SHORT }}

      - name: Push latest tag
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/my-app:latest

      - name: Push SHA tag
        run: |
          docker push ${{ secrets.DOCKER_USERNAME }}/my-app:sha-${{ env.SHA_SHORT }}
```
<img width="1851" height="875" alt="image" src="https://github.com/user-attachments/assets/3a325782-f9ef-471d-96d8-0a447e596a45" />

**Verify:** Go to Docker Hub — is your image there with both tags?
<img width="1845" height="718" alt="image" src="https://github.com/user-attachments/assets/cc253a41-404f-4924-ab0d-7372fa612b99" />

---

### Task 4: Only Push on Main
Add a condition so the push step only runs on the `main` branch — not on feature branches or PRs.
```
- name: Push latest tag
  if: github.ref == 'refs/heads/main' # add this condition
  run: docker push ${{ secrets.DOCKER_USERNAME }}/my-app:latest

- name: Push SHA tag
  if: github.ref == 'refs/heads/main'
  run: docker push ${{ secrets.DOCKER_USERNAME }}/my-app:sha-${{ env.SHA_SHORT }}
```
```
on:
  push:
    branches:
      - '**'
```
```
git checkout -b feature/test
git commit -m "test"
git push origin feature/test
```

Test it: push to a feature branch and verify the image is built but NOT pushed.
<img width="1851" height="875" alt="image" src="https://github.com/user-attachments/assets/aa6efae8-3398-4bb1-95ba-5cd8112d09e3" />

---

### Task 5: Add a Status Badge
1. Get the badge URL for your `docker-publish` workflow from the Actions tab

<img width="298" height="307" alt="image" src="https://github.com/user-attachments/assets/820e0d48-e3f0-498c-ab21-d9e721930a8e" />

<img width="592" height="482" alt="image" src="https://github.com/user-attachments/assets/77161fb4-98b8-45e1-bd00-17b06d94ff91" />

2. Add it to your `README.md`
```
[![Build and Push Docker Image](https://github.com/bumesh7/github-actions-practice/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/bumesh7/github-actions-practice/actions/workflows/docker-publish.yml)
```
```
git add README.md
git commit -m "Added workflow status badge"
git push
```
3. Push — the badge should show green

```
Green -Pass
Red - Fail
```

<img width="1057" height="237" alt="image" src="https://github.com/user-attachments/assets/05e29ab8-1f10-4d55-98b5-525eefe304f4" />

---

### Task 6: Pull and Run It
1. On your local machine (or a cloud server), pull the image you just pushed
```
docker login -u umesh4999

enter password
```
<img width="1213" height="288" alt="image" src="https://github.com/user-attachments/assets/831c3919-b84d-4fcf-9dab-5114c05a8aa6" />

2. Run it
```
docker run -d -p 5000:5000 umesh4999/my-app:latest
```
3. Confirm it works
<img width="552" height="249" alt="image" src="https://github.com/user-attachments/assets/fb3ca875-cdb4-4479-87ff-3ca2d809013b" />

Write in your notes: What is the full journey from `git push` to a running container?

---

