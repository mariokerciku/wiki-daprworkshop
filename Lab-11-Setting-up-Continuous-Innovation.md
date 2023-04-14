In this lab, you will add the capability to push every commit you did to the repository to be deployed to your test or production servers.
## Step 1: Install GitHub Commandline
To run workflows manually, we can use the GitHub command line interface.
To install this, you can run the following script in a new terminal bash window:
```bash
type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```


## Step 2: setting up a hosted runner to enable deployment in your local codespace minikube
Normally you deploy to a cluster that is either reachable from a GitHub Hosted runner or from a runner that you have running locally that can reach the servers where you want to deploy. In our Lab setup, we run a Kubernetes cluster on our local codespace development box, which does not provide access from the outside world. The only way we can deploy to that minikube instance is by having a GitHub Actions runner, running on the codespace dev container.

This is why we are starting with adding a hosted runner to our dev container.

To get the runner on your local dev container, take the following steps.
Go to the settings page of the repository.
In this settings page, go to actions, runners, as shown in the below picture.

<img width="970" alt="image" src="https://user-images.githubusercontent.com/3602709/231933903-9e172ff2-20d2-4bd7-871f-15002ca97d79.png">

Here click on the new runner button in the top right corner.
This will result in the following page, where we select Linux as the operating system and x64 as the architecture
<img width="883" alt="image" src="https://user-images.githubusercontent.com/3602709/231934188-c83cc072-d8f4-423f-a1be-22a082010591.png">

Now follow the steps as shown on the page to create and start the self-hosted runner.

The moment the configuration step asks for the name of the runner, provide a name that can be related to you, like your name
the next step, you can define additional labels you want to give to your runner. This can be used to pin an action workflow to run on a specific runner. We like to use this, so we can select our own runner for our own workflows. Add an additional label that has the following notation <firstname>-<lastname>

When all configuration is done and you have started the runner, you should see the following message:
``` cmd
√ Connected to GitHub

Current runner version: '2.303.0'
2023-04-14 03:36:56Z: Listening for Jobs
```

## Step 3: Creating a CI/CD actions workflow
In this step, you create the workflow that first creates the container images and then runs a security validation, and if there are no critical security issues, deploys the newly created image to the MiniKube cluster.

Create a new file with the name ci-cd-frontend.yaml and place this in a folder called `.github/workflows/`
Copy the following yaml into this file:
```yaml
name: Build and deploy frontend

on:
  workflow_dispatch:
  push:
    branches: [ "mygithubhandle-automation" ]
    paths:
    - 'frontend/**'
jobs:

  build:

    runs-on: ["selfhosted", "marcel-devries"]
    env:
      imageRepository: 'frontend'
      containerRegistry: 'psgloboticket.azurecr.io'
      dockerfilePath: 'frontend/Dockerfile'
      deploymentFile: 'frontend.yaml'
      namespace: 'globoticket'

    steps:
    - uses: actions/checkout@v3

    - name: Build an image from Dockerfile
      run: |
        docker build -t ${{env.imageRepository}}:${{github.run_number}} -f ${{env.dockerfilePath}} ${{github.workspace}}
        
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: '${{env.imageRepository}}:${{github.run_number}}'
        format: 'sarif'
        output: 'trivy-results.sarif'
        
    - name: Upload Trivy scan results to GitHub Security tab
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

    - name: Replace tokens
      uses: cschleiden/replace-tokens@v1.0
      with:
        files: ${{github.workspace}}/lab-resources/kubernetes/${{env.deploymentFile}}
      env: 
        Build.BuildId: ${{github.run_number}}
    - name: Upload a Build Artifact
      uses: actions/upload-artifact@v2.2.4
      with:
        name: deployfile
        path: ${{github.workspace}}/lab-resources/kubernetes/${{env.deploymentFile}}
        
  deploy:
    runs-on:  ["selfhosted", "marcel-devries"]
    needs: build
    steps:
    - name: Download artifact from build job
      uses: actions/download-artifact@v2
      with:
        name: deployfile
          
    - uses: Azure/k8s-deploy@v4
      with:
        namespace: 'globoticket'
        manifests: |
            ./${{env.deploymentFile}}
```

After saving this file, we need to commit this to the GitHub repository to execute the actions.
We need to push this to the branch name we denoted in the YAML file. In my case, this is `vriesmarcel-automation`, but this should be a unique branch name with your GitHub handle instead of mine.

The moment you have pushed the branch, you can go to the actions tab and see if the workflow is running on your runner.



