name: CI/CD Pipeline Aplikasi Kampus
on:
push:
branches: [ main ]
jobs:
kualitas-dan-keamanan:
runs-on: ubuntu-latest
steps:
- uses: actions/checkout@v3
- name: Automated Unit Testing
run: mvn test
- name: Static Code Analysis
uses: sonarsource/sonarqube-scan-action@master
env:
SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
- name: Vulnerability Scan
uses: aquasecurity/trivy-action@master
pembungkusan-dan-rilis:
needs: kualitas-dan-keamanan
runs-on: ubuntu-latest
steps:
- uses: actions/checkout@v3
- run: docker build -t ghcr.io/kampus/app-akademik:${{ github.sha }} .
- run: docker push ghcr.io/kampus/app-akademik:${{ github.sha }}
penyebaran-produksi:
needs: pembungkusan-dan-rilis
runs-on: ubuntu-latest
steps:
- name: Deploy via Helm
run: helm upgrade --install app-kampus-live ./charts/app-akademik
