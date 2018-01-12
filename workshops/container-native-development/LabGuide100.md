# Containerize Your Java Microservice

![](images/100/header.png)

## Introduction

This is the first of several labs that are part of the **Oracle Public Cloud Container Native Development workshop.** This workshop will walk you through the process of moving an existing application into a containerized CI/CD pipeline and deploying it to a managed Kubernetes service in the Oracle Public Cloud.

You will take on 2 personas during the workshop. The **Lead Developer Persona** will be responsible for configuring the parts of the automated build and deploy process that involve details about the application itself. The **DevOps Engineer Persona** will configure the parts of the automation involving the Kubernetes infrastructure. To containerize and automate the building and deploying of this application you will make use of Wercker Pipelines, Oracle Container Registry, and Oracle Container Engine.

During this lab, you will take on the **Lead Developer Persona** and work on containerizing your existing Java application. You will set up Wercker to monitor your application's source code repository for commits and automatically trigger a build, test, and package pipeline, which will result in a Docker image of your application that is ready to be deployed.

**_To log issues_**, click here to go to the [GitHub oracle](https://github.com/oracle/learning-library/issues/new) repository issue submission form.

## Objectives
**Containerize Your Java Application and Automate Building and Packaging**
- Create Wercker Application
  - Fork Java Application on GitHub
  - Create a Wercker account
  - Create Wercker application
- Create and Run Wercker Pipelines
  - Configure Pipelines and Workflow in Wercker
  - Define Wercker Build Pipeline
  - Set Environment Variables in Wercker
  - Define Wercker Publish Pipeline
  - Validate Workflow Execution

## Required Artifacts
- The following lab requires:
  - an Oracle Public Cloud account that will be supplied by your instructor.
  - a [GitHub account](https://github.com/join)

# Containerize Your Java Application and Automate Builds

## Create Wercker Application

### **STEP 1**: Fork Java Application on GitHub
- From any browser, go to:

    [https://github.com/derekoneil/twitter-feed]()

- Click **Fork** in the upper right hand corner of the browser. **Sign in** if prompted.

  ![](images/100/1.png)

  **NOTE:** If prompted, choose to fork the repository to your account (this may occur if your account is also a member of an organization on GitHub).

### **STEP 2**: Create a Wercker account
  **NOTE** If you already have a Wercker account, proceed to **STEP 3**.

- In a new browser tab, go to:
    [http://www.wercker.com/]()

- Click **Sign Up** in the upper right hand corner of the browser. Alternately, if you have already signed up for a Wercker account, click **log in** and then **log in with GitHub** and skip to **STEP 3**.

  ![](images/100/2.png)

- Click **Sign Up Using GitHub**

  ![](images/100/3.png)

- Click the green **Authorize Wercker** button

  ![](images/100/4.png)

- Enter **a username and your email address** to complete your Wercker account creation

  ![](images/100/5.png)

### **STEP 3**: Create a Wercker Application

- If this is your first Wercker application, click the blue **Create your first application** button. If you already have applications in your Wercker account, click the **plus button** in the upper right hand corner of the browser and select **Add application**:

  ![](images/100/6.png)


- Leave the default selections of your account and GitHub and click **Next**

  ![](images/100/7.png)

- Click on the **twitter-feed** repository that appears in the list of your GitHub repositories, then click **Next**

  ![](images/100/8.png)

- Leave the default selection of checkout without an SSH key and click **Next**

  ![](images/100/9.png)

- Click **Create**

  ![](images/100/10.png)

- Do not generate a wercker.yml file -- we will create one in a later step.

## Create and Run Wercker Pipelines

### **STEP 4**: Configure Pipelines and Workflow in Wercker

- Navigate to the Wercker page for your newly-created application (you will already be on that page if you just completed **STEP 3**). Notice that you are viewing the **Runs** tab. This is where any executions of your workflow will be recorded.

  ![](images/100/16.png)

- Click the **Workflows** tab. You will see that Wercker has created the beginning of a workflow for you already. This workflow is triggered by a Git commit and it will execute a pipeline called **build**.

  ![](images/100/17.png)

- The **build** pipeline will be used to build and unit test our application. Let's create a new pipeline to store the resulting Docker image in the Oracle Container Registry. Click the **Add new pipeline** button.

  ![](images/100/18.png)

- Fill in **push-release** for the name of the pipeline and the YML name of the pipeline and click **Create**.

  ![](images/100/19.png)

- Now that you've created the pipeline, click on the **Workflows** tab to return to the workflow editor. Click the **plus sign** next to the build pipeline in the editor.

  ![](images/100/20.png)

- In the **Execute Pipeline** drop down list, choose the pipeline we just created, **push-release**. Leave the other fields at their default values and click **Add**.

  ![](images/100/21.png)

- Now that we've got a workflow configured that will build and store a Docker image containing our application, we need to define exactly how to do that in a file called **wercker.yml**, which we will store in our application's Git repository.

### **STEP 5**: Define Wercker Build Pipeline

- Switch back to your GitHub browser tab, showing your forked copy of the **twitter-feed** repository, and click **Create new file**

  ![](images/100/13.png)

- In the **Name your file...** input field, type **wercker.yml**

  ![](images/100/14.png)

- In the **Edit new file** input box, **paste** the following:

```yaml
#Use OpenJDK base docker image from dockerhub and open the application port on the docker container
box:
  id: openjdk
  ports:
    - $PORT

#Build our application using Maven, just as we always have
build:
  steps:
    - install-packages:
        packages: maven
    - script:
        name: maven build
        code: mvn clean assembly:assembly
```

- You should have **14 lines** of YAML in the editor:

  ![](images/100/22.png)

- Let's look at the two sections of YAML that we've just added. The first section describes a **box**. A **box** is the image that you want Wercker to pull from a Docker registry (the default is Docker Hub) to build your pipeline from. In our case, we need a Java environment to run our application, so we will pull the **openjdk** image from Docker Hub.

  The second section defines our first pipeline, the **build** pipeline. Our **build** pipeline consists of two **steps**, one to install any packages required for our build process (**install-packages**), and one to execute a shell command to invoke the build (**script**).

  Since our application is built using Maven, we will instruct Wercker to install the Maven package first, using the **install-packages** step. Then we'll build the application the same way we would normally, invoking Maven in the **script** step.

- At the bottom of the page, click the **Commit new file** button.

  ![](images/100/23.png)

- Switch back to your Wercker browser tab and click on the **Runs** tab. If you are quick enough, you will see that Wercker has been notified of your new Git commit (via a webhook) and is executing your workflow.

  ![](images/100/24.png)

- You should see that the **build** pipeline completes successfully, but the **push-release** pipeline fails. That's what we expected, since we have not yet defined the steps for the **push-release** pipeline in our wercker.yml file.

  ![](images/100/25.png)

- Click on the green **build** pipeline to drill into the details of each step. Note that you can click on each step to see the console output produced by that step. In our case that output includes things like the results of the tests that Maven executed before packaging our application. If any commands produce an error status code, Wercker will abort the workflow and notify you via email.

  ![](images/100/26.png)

- Our next step is to define the second part of our workflow, the **push-release** pipeline, which will store our container image in the Oracle Container Registry after a successful **build**. This pipeline will make use of some environment variables, so let's get those set up first.

### **STEP 6**: Set Environment Variables in Wercker

- In your Wercker browser tab, click the **Environment** tab.

  ![](images/100/31.png)

- Create an environment variable by filling in the **Key** and **Value** boxes and clicking **Add**. Repeat this step for each variable listed below.
  ![](images/100/32.png)

```
  Key:              Value:
  DOCKER_EMAIL      Your email address
  DOCKER_USERNAME   Your Github username
  DOCKER_REGISTRY   wcr.io
  DOCKER_REPO       wcr.io/your-github-username/twitter-feed
  PORT              8080
```

**NOTE**: The Docker email and username variables are required to authenticate to the container registry.

### **STEP 7**: Define Wercker Publish Pipeline

- Switch to your Github browser tab, click on the **wercker.yml** file, and click the **pencil icon** to begin editing.

  ![](images/100/27.png)

- After the definition of the build pipeline, **paste** the following YAML:

```yaml
#Push the docker image with our built and tested application to the Oracle Container Registry
push-release:
  steps:
    - internal/docker-push:
        tag: $WERCKER_GIT_BRANCH-$WERCKER_GIT_COMMIT
        working-dir: /pipeline/source
        ports: $PORT
        cmd: sh target/bin/start
```

  ![](images/100/28.png)

- This section of YAML defines our second pipeline, **push-release**. It uses a type of step we haven't seen before, **internal/docker-push**. This is a command developed by Wercker, which will take the current pipeline image and push it to a Docker registry. You can, however, use a different repository -- the step supports a `repository` parameter. You can find more info in the [documentation](http://devcenter.wercker.com/docs/steps/internal-steps#docker-push).

  There are two more things to note about this pipeline definition. First, the `cmd` parameter tells Docker how to start our application. Note that this is just the start script generated by Maven, exactly as we would have started it outside of Docker.

  Second, we have made use of several environment variables. We are tagging the release with the Git branch name and the Git commit hash so that we can trace our running application version back to a specific Git commit. These two variables (along with [others](http://devcenter.wercker.com/docs/environment-variables/available-env-vars#hs_cos_wrapper_name)) are automatically made available by Wercker. The `$PORT` environment variable, on the other hand, references the one that we defined ourselves in the previous step.

- At the bottom of the page, click **Commit changes**.

  ![](images/100/29.png)

### **STEP 8**: Validate Workflow Execution

- Switch to your Wercker browser tab and click the **Runs** tab within Wercker. You'll see the workflow executing as a result of your Git commit.

  ![](images/100/30.png)

- Once the workflow finishes, you'll see both the build and push-release pipelines turn green to indicate success. After that happens, click on the **Releases** button in the top navigation menu in Wercker.

  ![](images/100/34.png)

- On the Releases page, you'll see that a repository for your application has been automatically created for you, and your **push-release** pipeline has saved your Docker image into the repository.

  ![](images/100/33.png)

- Click on the **title** of the repository (yourusername/twitter-feed) to view the history of published images (you'll only have one at this point). Note that when you hover over the **Download** icon, the `docker pull` command to pull this image is displayed. At this point you could run your application in Docker on your laptop, tie in to your existing deployment setup, or (as you'll see in the next lab) deploy it to **Oracle Container Engine**--our managed Kubernetes cloud service.

  ![](images/100/35.png)

**You are now ready to move to the next lab.**