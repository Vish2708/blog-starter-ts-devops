# Neverbland Junior DevOps Challenge
## The Solution:
Continuous integration/continuous deployment is a software engineering practice that helps teams to collaborate better and improve their overall software. With GitHub Actions, you can easily integrate this into your GitHub project without using an external platform.

Advantages of CI/CD:

-Fault isolation is simpler and faster. Since changes are smaller, it is easier to isolate the changes that cause a bug after deployment. This makes it easier to fix or roll back changes if necessary
-Since CI/CD encourages small, frequent changes, code review time is shorter
-A major part of the CI/CD pipeline is the automated testing of critical flows for a project. This makes it easier to prevent changes that may break these flows in production
-Better code quality is ensured because you can configure the pipeline to test against linting rules.

**PROJECT WORKFLOW**

![image](https://user-images.githubusercontent.com/88805794/150747799-2c03468a-ee9d-42c7-9ddd-2992d73c7d9e.png)

# STEPS FOR THE TASK ( CONTINOUS INTEGRATION )
1. Setting up the project : installing all the dependencies we need
2. Creating the application and adding the tests
3. Setting up the workflow: the workflow.yaml file will build the application and runs tests whenever a pull request is merged to the main branch and deploy the application to EC2 instance.

**workflow.yaml**

1. Triggers the work flow on push or pull request

 **push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    
2. Workflow run is made up of one or more jobs that can run sequentially or in parallel

  **jobs:
  continuous-integration:

3. The type of runner that the job will run on

    **runs-on: ubuntu-latest

4. Steps represent a sequence of tasks that will be executed as part of the job

    **steps:
  
5. Checks out your repository under $GITHUB_WORKSPACE, so your job can access it

    - uses: actions/checkout@v2
    - uses: actions/setup-node@v2
      with:
        node-version: '12.13.0'
    - run: npm install
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
         aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
         aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         aws-region: eu-west-2
    - run: npm run build --if-present
    - run: npm test
    
    
# Steps for the task ( CONTINOUS DEPLOYMENT)

1. In AWS Management console create a IAM role.
   One for EC2 instance role
   One for Codedeploy service role
2. Create and launch EC2 instance
3. Configure AWS CodeDeploy Service
4. Running the complete pipepline
    
    - continuous-deployment:
    - runs-on: ubuntu-latest
    # Before we deploy anything, we are going to upload our previously downloaded ssh private key, into Github secrets so that we can reference it in our CI pipeline.
    - needs: [continuous-integration]
    - if: github.ref == 'refs/heads/main'
    - steps:
     # Configure AWS credentials with secrects for security 
      - name: Configure AWS credentials
      - uses: aws-actions/configure-aws-credentials@v1
      - with:
         - aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
         - aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
         - aws-region: eu-west-2
    # CodeDeploy Deployment
      - name: Create CodeDeploy Deployment
      - id: deploy
      - run: |
         - aws deploy create-deployment \
            --application-name Neverbland \
            --deployment-group-name Neverbland \
            --deployment-config-name CodeDeployDefault.OneAtATime \
            --github-location repository=${{ github.repository }},commitId=${{ github.sha }}**
            
            

## Notes

This repo uses the Next.js example [blog-starter-typescript](https://github.com/vercel/next.js/tree/canary/examples/blog-starter-typescript) project.

This blog-starter-typescript uses [Tailwind CSS](https://tailwindcss.com). To control the generated stylesheet's filesize, this example uses Tailwind CSS' v2.0 [`purge` option](https://tailwindcss.com/docs/controlling-file-size/#removing-unused-css) to remove unused CSS.

[Tailwind CSS v2.0 no longer supports Node.js 8 or 10](https://tailwindcss.com/docs/upgrading-to-v2#upgrade-to-node-js-12-13-or-higher). To build your CSS you'll need to ensure you are running Node.js 12.13.0 or higher in both your local and CI environments.
