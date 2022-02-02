# CI/CD with React

##### [Medium Article](https://medium.com/front-end-weekly/ci-cd-with-react-f4af73618d57)

![](https://i.ibb.co/jgzBRmv/1-vaa-ZAt34-HCAc8-TPd-XYdj-Q.png)

## What is CI/CD?

### A beginner's guide to CI/CD for React?

Continuous Integration (CI) is a development practice that helps ensure that software components work together. Continuous delivery (CD) is the ability to deploy your integrated code into production without the need for human intervention.

Let’s start building with the very basic TODO application and will see how we can write unit tests and then we will integrate it with Circle CI which is one of the most popular CI tools.

## Prerequisites

We are going to use the React testing library in order to write the unit tests. For this purpose, if you don’t have any project you may clone this repository from [here](https://github.com/Abdullah-waqas/react-todo-list-app) which has a pre-written unit test of the todo list application.

## What we will cover?

In this article, we will cover how to set up CI/CD for a react application. The tools that we will use to setup are:

- **Circle CI** which will help us to build a pipeline.
- **Codecov** to generate the code coverage report.
- **Netlify** a free hosting server to host static web applications.

## Scenario

We want when developers push the code to Github, a pipeline should execute all unit tests, generate code coverage reports, the status should be available on the PR after it is created.
And it should be deployed to Netlify when the PR is merged.

## Clone Github Repository

Now, Let’s clone the [Github](https://github.com/Abdullah-waqas/react-todo-list-app/tree/initial) repository. Initially, you will find the structure of the application should look something like this
![](https://i.ibb.co/NCct6j1/1-05avld-Ngl-XBTIBqsa-CMo-UA.png)

Run test command to make sure tests run correctly

```
yarn test
```

![](https://i.ibb.co/pXNL1dr/1-JIli-BWa-BH7-Fqi1-Q-V2z2-HA.png)

## Setting Up CI With CircleCI

In order to set up, you must have an account on CircleCI if you have already one that’s great. Log in, and go to the project dashboard and click on **Setup Up Project**

![](https://i.ibb.co/hsS2sq6/1-d8-T1qr-J7-CCr-TOzial9-Gpj-A.png)

After selection, it will ask you to provide a config.yml file.

![](https://i.ibb.co/F8Fw2bP/1-S82-WTn1-MSa-ITIf-as-ROvg.png)

Let’s create a folder with the name **.circleci** and add a **config.yml** file in it.

```yml
version: 2.1
orbs:
  node: circleci/node@3.0.0
jobs:
  build:
    docker:
      - image: node:11.10.1

    working_directory: ~/repo

    steps:
      - checkout

      # Download and cache dependencies
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package.json" }}
            # fallback to using the latest cache if no exact match is found
            - v1-dependencies-

      - run: npm install

      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

      # run tests!
      - run: npm run test
```

**version:** specify circle ci version.

**docker:** specify a docker image used to create the container of the environment.

**checkout:** used to check out the code from the current branch into the working directory.

**run:** execute a bash command.

**save_cache:** used to store a cache of files/directory in the CircleCI storage. i.e it installs the npm dependencies only once and then cache the <i>node_modules</i> folder using a hash of the package.json file of your project as the cache key.

**restore_cache:** used to restore cache file using cache key.

**command:** where your tests run.
Now commit and push it on the main branch, then select **Use Existing Config** and select **Start Building**

![](https://i.ibb.co/s3SNyqC/1-z-LNj-FMD5r-MA7-CHnta-Bx-Q.png)

Select **Start Building**

![](https://i.ibb.co/Lv9N8KT/1-3d73-9-JPtw3k3l-Nl-Uu0-IBQ.png)

After selection, you will see all your tests start passing on circle CI.

![](https://i.ibb.co/bmNz5sc/1-4g-YV5-Ni-Bc-SBo-Ms0o9-N4-Zxg.png)

## Setup Codecov

In order to setup **codecov**, we have to do a few configurations in our existing codebase. We need to add a test coverage command which will generate the report of tests. Add the following commands in the **package.json** file.

```
"test:coverage": "cross-env CI=true npm test -- --env=jsdom --coverage",
"upload:test-report": "./node_modules/.bin/codecov",
```

and in config.yml let’s add the orbs

```
codecov: codecov/codecov@1.0.2
```

Why we need orbs? because it will upload our coverage report without dealing with complex configuration. Add a step in **config.yml** to upload the coverage report.

```
- codecov/upload:
  file: './coverage/clover.xml'
```

After the edit, the config.yml should look something like this

```yml
version: 2.1
orbs:
  node: circleci/node@3.0.0
  codecov: codecov/codecov@1.0.2
jobs:
  build:
    docker:
      - image: node:11.10.1

    working_directory: ~/repo

    steps:
      - checkout

      # Download and cache dependencies
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package.json" }}
            # fallback to using the latest cache if no exact match is found
            - v1-dependencies-

      - run: npm install

      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

      # run tests!
      - run: npm run test

      - run: npm run build

      - run: npm run test:coverage

      - codecov/upload:
          file: './coverage/clover.xml'
```

Now signup quickly to codecov and link the current repository
![](https://i.ibb.co/sCSpBm4/1-WOv-L4v-Bso-Cg-Sg-Ew-I-VXZw.png)

Go to **Add new repository**, search and select your repository.

![](https://i.ibb.co/YcygvH5/1-N9g-Np-n-Tc-Iw-CDBpo3jbtg.png)

Copy the **token** and go to the **Project Settings** of your project in **CircleCI**
![](https://i.ibb.co/xmnRmjn/1-4ouz-Nfucg-P3-P3h-o146-Jo-A.png)

Select **Environment Variables** and Add Environment Variable.
![](https://i.ibb.co/dBK71ys/1-ql-XXKxu-KWUVq-WPCSjdk-Xo-A.png)

Now commit the code and make a pull request and wait a few minutes to execute all the checks. You will see a codecov report in your PR.
![](https://i.ibb.co/qkp9RXB/1-6-BERrh39-UGS2ruo3ga-D0-Q.png)

## Setup Deployment on Netlify

In the final step, we will deploy the application on netlify and will make it a part of our pipeline. First of there should be netlify-cli needs to be installed. Since I am using yarn I can install by typing the following command otherwise you can use npm too.

```
yarn add netlify-cli
```

After installation create a deployment script to deploy the app

```
"netlify:deploy": "netlify deploy --dir=./build -p -m \"$(git log -1 --pretty=%B)\""
```

Now navigate to your config.yml file add the deployment step

```
- run: npm run netlify:deploy
```

Now we need to signup on netlify and connect your GitHub, then we need to copy **NETLIFY_AUTH_TOKEN** and **NETLIFY_SITE_ID** from netlify and add it as an environment variable in **CircleCI**. You may look at the following screenshots for the steps.

![](https://i.ibb.co/3khntp6/1-u3lmz3-ju-uz-K7a-EJg-Zw.png)
![](https://i.ibb.co/mC4g9hM/1-O0-W6m0mv-p-Gah-Zdn-X5z-Lw.png)
![](https://i.ibb.co/5nyxNHM/1-CSpvya-Hg-Oewn-Xod-Ljr-Xt1-Q.png)

Now navigate to the user setting generate the access token and copy that access token.

![](https://i.ibb.co/TRsTQvt/1-cp-Z3-DPIGAxv-FZr2-JPKIuaw.png)

In your CircleCI go to project settings and add it as NETLIFY_AUTH_TOKEN and paste the value as we did for codecov.

![](https://i.ibb.co/sW3yY4z/1-8s-Nm3265-EFd-REVNmc-cykg.png)

Now select the site overview in your Netlify dashboard and select **Site Settings**
![](https://i.ibb.co/0cdxc5b/1-ab015f-RNPe3-Nfcr-Gn-Ae-MDw.png)

Copy that API ID

![](https://i.ibb.co/dGNz68b/1-l0-HPGk-FXH-a2sb1-FLw-Siog.png)

Add it as environment variable with name NETLIFY_SITE_ID

![](https://i.ibb.co/xMX5rZs/1-3w-GUBDs-X1-WMPBau-FOg-Nbqg.png)

Whenever the pipeline is executed it will get the Token and Site Id from the Environment variable so we don’t need to pass it from package.json which is of course not secure. Now push your code and it will deploy your changes on netlify once you merge your changes on the main branch.
All is done! The final version of the code can be accessed here.
**Happy Coding!** 🎉 🥳
