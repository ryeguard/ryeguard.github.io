# Deploying a Next.js App to Google Cloud Run using Docker

In this post, we will go through how you can get started with setting up a [Next.js](https://nextjs.org/) app. We will then deploy the app to [Google Cloud Run](https://cloud.google.com/run) using [Docker](https://www.docker.com/). We will also set up a [Cloud Build](https://cloud.google.com/build) trigger so that a new version of our app is automatically built and deployed every time we push to the `main` branch of our repository.

## Table of Contents

- [Getting Started](#getting-started)
- [Deploying to Google Cloud](#deploying-to-google-cloud)
  - [1. Prerequisites](#1-prerequisites)
  - [2. Reading Up](#2-reading-up)
  - [3. Preparing Locally](#3-preparing-locally)
  - [4. Preparing a Github Repository](#4-preparing-a-github-repository)
  - [5. Configuring Google Cloud](#5-configuring-google-cloud)
  - [6. Testing](#6-testing)
- [Configuring a Custom Domain](#configuring-a-custom-domain)

## Getting Started

If you, like me, have very little experience with web development, you might be wondering what Next.js is and where it sits in the web development ecosystem. Next.js is a framework for building [React](https://reactjs.org/) apps. In turn, React is a [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) library for building user interfaces. React is a very popular library and is used by both large companies and individuals. React is a powerful library, but it is also lower level and unopinionated. It is flexible, but it also means that you have to make a lot of decisions yourself. Next.js is a framework that builds on top of React and provides a lot of functionality out of the box. This makes it easier to get started with building a React app, and it also makes it easier to build a React app that is fast, accessible, and SEO-friendly.

I strongly recommend to start with the [Next.js Learn tutorials](https://nextjs.org/learn/) to get a better understanding of what Next.js is and how it works. The tutorial is very well constructed and will walk you though building a simple Next.js app. You will even deploy an app to [Vercel](https://vercel.com/), which is the company behind Next.js.

If you're not familiar with React either, I recommend to start with the [Learn React](https://react.dev/learn) tutorial first. It is also very well constructed and will give you a good understanding of how React works.

## Deploying to Google Cloud

### 1. Prerequisites

- A [Google Cloud](https://cloud.google.com/?hl=en) account.
- A [Github](https://github.com) account.
- [Docker](https://docs.docker.com/get-docker/) installed on your machine.

### 2. Reading Up

The best place to start is with the official Next.js documentation. On the [Deploying](https://nextjs.org/docs/pages/building-your-application/deploying) page we learn that Next.js apps can be deployed to any hosting provider that supports Docker containers. This is great news, because it means that we can deploy to Google Cloud Run. The [Google Cloud Run documentation](https://cloud.google.com/run/docs) is also a great resource. It is a lot of information, but we will only need a small part of it which I am to explain here.

[This blog post](https://www.frontendeng.dev/blog/6-deploying-nextjs-app-on-cloud-run-ci-cd#setup-your-repository-on-github) by [Frontend Engineer](https://www.frontendeng.dev/) is also a great resource. We will be going through similar steps.

### 3. Preparing Locally

Both under [Deploying/Docker Image](https://nextjs.org/docs/pages/building-your-application/deploying#docker-image) and [Deploying/Managed Server](https://nextjs.org/docs/pages/building-your-application/deploying#managed-server) we are linked to the [vercel/next.js repository](https://github.com/vercel/next.js/tree/canary/examples/with-docker) `with-docker` examples.

Feel free to create your app with the `create-next-app` commands provided, or follow the instructions under the "In existing projects" section. I will not repeat the steps here, but I will add some additional information that I found useful.

- In an existing project, copy _both_ the `Dockerfile` _and_ the `.dockerignore` files from the `with-docker` example into the root of your project.

- In an existing project, if you do not have a `next.config.js` file, create it in the root of your project with the following contents:

```js
/** @type {import('next').NextConfig} */
module.exports = {
  output: "standalone",
};
```

- Make sure the image builds and runs locally before moving on to the next steps. You can do this by running the commands provided in the `with-docker` README:

```bash
# Build the image
docker build -t nextjs-docker .
# Run the container
docker run -p 3000:3000 nextjs-docker
```

where `nextjs-docker` is the name/tag of the image. You can choose any name you want.

### 4. Preparing a Github Repository

If you haven't already, create a new repository on Github and push your code to it. This should be straight forward if you have used Github before. If you haven't, you can follow the instructions on the [Github Docs](https://docs.github.com/en/get-started/quickstart/create-a-repo).

> ⚠️ Note: Make sure you do not push any sensitive information to Github. This includes any `.env` files or any other files that contain secrets. This is easily avoided using a `.gitignore` file.

### 5. Configuring Google Cloud

The [next.js repo's `with-docker` example](https://github.com/vercel/next.js/tree/canary/examples/with-docker) provides instructions on how to deploy using the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install). However, we will use the [Google Cloud Console](https://console.cloud.google.com/) to set up our project, [Cloud Run](https://cloud.google.com/run) service, [Cloud Build](https://cloud.google.com/build) trigger, and Github connection. This way, a new version of our app will be automatically built and deployed every time we push to the `main` branch of our repository.

Most of the following steps should be straight forward, I will provide some additional information where I think it is useful.

1. Create a new project in the Google Cloud Console. You can follow the instructions on the [Google Cloud Docs](https://cloud.google.com/resource-manager/docs/creating-managing-projects).

1. Enable billing for your project by searching for "Billing" and following the steps. You can follow the instructions on the [Google Cloud Docs](https://cloud.google.com/billing/docs/how-to/modify-project).

1. Navigate to [Cloud Run](https://console.cloud.google.com/run) (can also be done by searching!) and click `Create service`. You can follow the instructions on the [Google Cloud Docs](https://cloud.google.com/run/docs/quickstarts/build-and-deploy#console). In the setup guide:

   - Select `Continuously deploy new revisions from a source repository`.

   - Click `Set up Cloud Build`, which will open a sidebar.

     - As "Repository provider" select `Github`.
     - You will be asked to install the Google Cloud Build app on Github. You can allow access to all repositories or only to selected repositories. To make things easier, I recommend to allow access to all repositories.
     - Select your repository from the drop down list which is now available.
     - Check the consent box and click `Next`.
     - As "Branch" use the default value `^main$`. This will trigger a build every time you push to the `main` branch.
     - As "Build type" select `Dockerfile` and enter the path to your `Dockerfile`, i.e., `/Dockerfile` in our case.
     - Click `Save`.

   - Choose a Service name, e.g. `nextjs-app`.
   - Choose a Region, preferably with Low CO2, e.g. `europe-north1` if you're in Stockholm like me.
   - Under "CPU allocation and pricing" we can tune some options to minimize cost:

     - Choose `CPU is only allocated during request processing`.
     - Set "Minimum number of instances" to `0`, as we will allow cold starts.
     - Set "Maximum number of instances" to `1`, as we don't expect a lot of traffic.

   - Set "Ingress Control" to `All` as we want to publish this app on the internet.
   - Set "Authentication" to `Allow unauthenticated invocations` as we are creating a public API/website.
   - Expand the "Container, Networking, Security" section:

     - Set "Container port" as `3000` as this is the port that our app is listening on.
     - Under "Environment variables" we can add any environment variables that our app needs:

   - Click `Create`.

1. Go to the [API Library](https://console.cloud.google.com/apis/library) and enable `Identity and Access Management (IAM) API`, so that we can trigger a deployment manually.

1. In the Service details page, click `Edit continuous deployment`.

   - Under "Configuration" make sure type is set to `Cloud Build configuration file (YAML or JSON)`.
   - As "Location" set `Inline`.
   - If you click the "Open editor" button, you should see some YAML that looks like the example below. If not, delete this trigger and create a new one.
   - Click `Save`.

1. Go to the [Cloud Build Triggers](https://console.cloud.google.com/cloud-build/triggers) page and click `Run` on the trigger you just created. This should add an entry to the "History" table. Click on the entry in the Build column to see the details. There should be 3 steps executing (Build, Push, Deploy), as seen in the YAML editor in the previous step. Once the build is complete, you should be able to see your app running on the internet. You can find the URL on the [Cloud Run Services](https://console.cloud.google.com/run) page.

   ```yaml
   steps:
   - name: gcr.io/cloud-builders/docker
       args:
       - build
       - "--no-cache"
       - "-t"
       - >-
           $_AR_HOSTNAME/$PROJECT_ID/cloud-run-source-deploy/$REPO_NAME/$_SERVICE_NAME:$COMMIT_SHA
       - .
       - "-f"
       - Dockerfile
       id: Build
   - name: gcr.io/cloud-builders/docker
       args:
       - push
       - >-
           $_AR_HOSTNAME/$PROJECT_ID/cloud-run-source-deploy/$REPO_NAME/$_SERVICE_NAME:$COMMIT_SHA
       id: Push
   - name: "gcr.io/google.com/cloudsdktool/cloud-sdk:slim"
       args:
       - run
       - services
       - update
       - $_SERVICE_NAME
       - "--platform=managed"
       - >-
           --image=$_AR_HOSTNAME/$PROJECT_ID/cloud-run-source-deploy/$REPO_NAME/$_SERVICE_NAME:$COMMIT_SHA
       - >-
           --labels=managed-by=gcp-cloud-build-deploy-cloud-run,commit-sha=$COMMIT_SHA,gcb-build-id=$BUILD_ID,gcb-trigger-id=$_TRIGGER_ID
       - "--region=$_DEPLOY_REGION"
       - "--quiet"
       id: Deploy
       entrypoint: gcloud
   images:
   - >-
       $_AR_HOSTNAME/$PROJECT_ID/cloud-run-source-deploy/$REPO_NAME/$_SERVICE_NAME:$COMMIT_SHA
   options:
   substitutionOption: ALLOW_LOOSE
   logging: CLOUD_LOGGING_ONLY
   substitutions:
   _PLATFORM: managed
   _SERVICE_NAME: nextjs-app # example service name
   _DEPLOY_REGION: europe-north1 # example region
   _TRIGGER_ID: 123ab4cd-ef5g-6hi7-8j90-klmnopq234r5 # example trigger ID
   _AR_HOSTNAME: europe-north1-docker.pkg.dev
   tags:
   - gcp-cloud-build-deploy-cloud-run
   - gcp-cloud-build-deploy-cloud-run-managed
   - nextjs-app # example service name
   ```

### 6. Testing

Finally, we can test that everything works as expected. Push a small change to the `main` branch of your repository. This should trigger a new build and deploy. You can follow the progress on the [Cloud Build History](https://console.cloud.google.com/cloud-build/builds) page. Once the build is complete, you should be able to see your app running on the internet.

## Configuring a Custom Domain

As previously stated in this guide, you are assigned a static URL when you create a Cloud Run service. This URL is not very user friendly, and you probably want to use your own domain. Documentation can be found in the [Cloud Run Docs](https://cloud.google.com/run/docs/mapping-custom-domains), but I will walk you through the steps here.

Go to the [Cloud Search Console](https://search.google.com/search-console) and add a property of type "Domain". I personally wanted to deploy the Next.js app to `app.rygard.se`, as opposed to `www.rygard.se`, so I entered `app.rygard.se` as (sub)domain.

The page will ask you to verify the ownership of this property via DNS record. Follow the instructions provided to add a TXT record in your host's DNS management page (Hostup.se in my case), much like we did in the [previous blog post](../231009_website/231009_website.md) for Github Pages.

Now, we can head to the [Cloud Run Services](https://console.cloud.google.com/run) page and click on our service. Find the [Domains page](https://console.cloud.google.com/run/domains) and add a new mapping. Select your service, e.g., `nextjs-app` and enter the domain we just verified, e.g. `app.rygard.se`. No need to enter anything the the subdomain field.

Now we are asked to make the final DNS changes. Head back to your hosts's DNS management page and add the records provided. These are four A records and four AAAA records.

Finally, allow some time for the change to propagate, after a while you should be able to reach your app on your custom domain, like `app.<domain-name>.se`.

<!--
## Bonus: Installing Google Cloud SDK

Even though this guide does not use the Google Cloud SDK, you might want to install it anyway. It is a very useful tool for interacting with Google Cloud. There are instructions on [Google Cloud Docs](https://cloud.google.com/sdk/docs/install). However, it requires a certain version of Python (3.8-3.10 at the time of writing). If you, like me, have a number of different Python versions installed, you need some slightly different setup.
-->
