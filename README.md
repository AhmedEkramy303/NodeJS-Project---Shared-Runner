# Node.js Project: CI/CD Pipeline to EKS Deployment

This project demonstrates a complete CI/CD pipeline for a Node.js application, from code commit to deployment on an Amazon Elastic Kubernetes Service (EKS) cluster, leveraging GitLab CI/CD.

## Table of Contents

1.  [Understanding Node.js](#1-understanding-nodejs)
2.  [Localhost Deployment and Testing](#2-localhost-deployment-and-testing)
3.  [Infrastructure Setup for Shared Runner](#3-infrastructure-setup-for-shared-runner)
4.  [Build and Test Stage](#4-build-and-test-stage)
5.  [SonarQube Stage](#5-sonarqube-stage)
6.  [Docker File Creation](#6-docker-file-creation)
7.  [Docker Build and Push Stage](#7-docker-build-and-push-stage)
8.  [Kubernetes Manifest File Creation](#8-kubernetes-manifest-file-creation)
9.  [Deployment Stage on EKS](#9-deployment-stage-on-eks)
10. [Project Summary](#10-project-summary)

---

## 1. Understanding Node.js

Node.js is an open-source, cross-platform JavaScript runtime environment that allows developers to execute JavaScript code on the server-side. It is built on Chrome's V8 JavaScript engine, enabling fast and efficient code execution.

### Key Uses:

* **Web Servers:** Building server-side applications and REST APIs.
* **Real-time Applications:** Developing interactive chat applications and other fast-processing systems.
* **Microservices:** Facilitating the creation of microservices architecture.

### npm (Node Package Manager)

npm is the default package manager for Node.js, crucial for managing packages, dependencies, and scripts.

### Common npm Commands:

* `npm init`: Initializes a project by creating a `package.json` file, which lists all project dependencies.
* `npm install`: Installs all dependencies listed in `package.json`.
* `npm run`: Helps in defining and running custom scripts (e.g., build, test).
* `npm publish`: Creates and publishes a package.

### Project Structure:

A typical Node.js project includes:

* `public/`: Contains web page content (e.g., HTML, images).
* `src/`: Holds the application's source code.
* `test/`: Contains test cases.
* `package.json`: Lists all project dependencies.
* `package-lock.json`: Specifies the exact versions of dependencies used in the project.
* `node_modules/`: Directory where installed npm packages are stored.
* `README.md`: Project documentation.

---

## 2. Localhost Deployment and Testing

Before deploying, it's essential to understand how the Node.js application behaves locally. Two scenarios are covered:

### Scenario 1: Raw Project (No `node_modules` or `package-lock.json`)

If the developer provides only the raw code, you need to build the project and install dependencies.

**Steps:**

1.  **Install Node.js and npm:** Follow the official Node.js installation steps for your operating system (e.g., Linux, Mac OS, Windows).
2.  **Verify Installation:** Check Node.js and npm versions using `node -v` and `npm -v`.
3.  **Install Dependencies:**
    * For `express` (for viewing in browser): `npm install express`.
    * For `jest` (for testing): `npm install --save-dev jest`.
4.  **Start the Server:** Run `npm start`. The application will typically run on port `3000`.

### Scenario 2: Already Built Project (with `node_modules` and `package-lock.json`)

If `node_modules` and `package-lock.json` are already present, you can directly start the server.

**Steps:**

1.  **Navigate to Project Directory.**
2.  **Start the Server:** Run `npm start`.

---

## 3. Infrastructure Setup for Shared Runner

The project utilizes GitLab managed runners, eliminating the need for self-hosted runners or agents.

### Components to Set Up:

* **GitLab Project:** Create a new public project in GitLab (e.g., "project Node"). Enable testing for development purposes.
* **SonarQube:**
    * Launch an EC2 instance (T3 medium or larger recommended due to SonarQube's resource requirements).
    * Install JDK on the EC2 instance (SonarQube is Java-based).
    * Create a dedicated `sonarqube` user and set a password.
    * Add the `sonarqube` user to the `sudoers` list.
    * Download and unzip the SonarQube zip file into the `/opt` folder.
    * Change the ownership of the SonarQube directory to the `sonarqube` user.
    * Switch to the `sonarqube` user.
    * Start SonarQube: Navigate to `sonarqube/bin/linux` and run the start script.
    * Access SonarQube on port `9000` (e.g., `http://your-ec2-public-ip:9000`).
    * Log in with default credentials (`admin`/`admin`) and update the password.
* **EKS Cluster:**
    * Create an EKS cluster (e.g., "Node.js cluster").
    * Choose the standard version for cost efficiency.
    * Utilize default VPC and security groups.
    * Select the latest available add-on versions to avoid future upgrades.
    * EKS cluster creation takes approximately 10-15 minutes.

---

## 4. Build and Test Stage

This stage focuses on building and testing the Node.js application using GitLab CI/CD.

### GitLab CI/CD Configuration (`.gitlab-ci.yml`):

* **Global Image:** Specify a Node.js image (e.g., `node:16`).
* **Stages:** Define `build` and `test` stages.
* **Build Job (`build_job`):**
    * Stage: `build`.
    * Script: `npm install` (to load dependencies).
* **Test Job (`test_job`):**
    * Stage: `test`.
    * Script: `npm test`.
* **SAST Job (Static Application Security Testing):**
    * Can be included to check for static vulnerabilities.
    * Set `allow_failure: true` for SAST and SonarQube jobs to prevent pipeline failure if these checks fail.

### Pushing Code to GitLab:

1.  Initialize Git: `git init`.
2.  Add files: `git add .`.
3.  Commit changes: `git commit -m "Initial project files"`.
4.  Add remote origin: `git remote add origin <your-gitlab-repo-url>`.
5.  Set upstream branch: `git branch -M main`.
6.  Push code: `git push -u origin main` (or `git push -u origin --all` if needed).
7.  Ensure the branch is *not* protected in GitLab settings before pushing.

Once the `.gitlab-ci.yml` is committed, the pipeline will automatically trigger, running the build and test stages.

---

## 5. SonarQube Stage

This stage integrates SonarQube for code analysis and quality checks.

### SonarQube Setup:

1.  **Configure GitLab Integration in SonarQube:**
    * Go to SonarQube settings and select "GitLab".
    * Provide a name (e.g., "Node.js") and your GitLab URL.
    * Generate a personal access token in GitLab with necessary permissions (e.g., `api`, `read_repository`, `write_repository`).
    * Paste the token into SonarQube and save the configuration.
2.  **Create SonarQube Project:** Select your GitLab project (e.g., "Project Node") and create a project in SonarQube using global settings.

### GitLab CI/CD Variables:

Add the following variables in your GitLab project's CI/CD settings (`Settings > CI/CD > Variables`):

* `SONAR_TOKEN`: Generate a new token in SonarQube for your project, copy its value, and set it as the variable value in GitLab. Uncheck "Protect variable" and check "Mask variable".
* `SONAR_HOST_URL`: Set the value to your SonarQube server URL (e.g., `http://your-ec2-public-ip:9000`). Uncheck "Protect variable" and leave "Mask variable" unchecked.

### SonarQube Project Properties (`sonar-project.properties`):

Create a file named `sonar-project.properties` in your repository's root with the necessary SonarQube properties.

### GitLab CI/CD Pipeline Update:

Add a `sonarqube` stage to your `.gitlab-ci.yml` file, including the SonarQube scanner commands.

After committing the changes, the pipeline will run, including the SonarQube analysis. The analysis results will be visible in the SonarQube dashboard.

---

## 6. Docker File Creation

A `Dockerfile` is essential for containerizing the Node.js application for deployment on Kubernetes.

### `Dockerfile` Contents:

Create a file named `Dockerfile` in your project's root:

```dockerfile
# Base Image: Use Node.js 16 as the base image
FROM node:16

# Set Working Directory: Define the application's working directory inside the container
WORKDIR /usr/src/app

# Copy Package Files: Copy package.json and package-lock.json to install dependencies
COPY package*.json ./

# Install Dependencies: Run npm install to install project dependencies
RUN npm install

# Copy Application Code: Copy all application code to the working directory
COPY . .

# Expose Port: Expose port 3000, which Node.js typically uses
EXPOSE 3000

# Start Application: Define the command to start the Node.js server
CMD [ "npm", "start" ]
