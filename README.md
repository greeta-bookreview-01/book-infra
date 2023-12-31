### AWS Startup Template For Spring Boot Developers
#### Welcome to AWS Full-Stack Developer Template: Swagger UI + Spring Boot + Terraform + Kubernetes + Keycloak Oauth2 Authorization Server + Github Actions + Local Docker Build and Start Environment + Integration Tests with TestContainers + Remote Debugging + Spring Cloud Gateway + Spring Cloud Stream + Dispatcher Pattern + AWS SSL Certificate + External DNS + AWS Load Balancer Controller + Spring Cloud Kubernetes + Grafana Observability Stack

**Keycloak Administration Console** will be available here: **https://keycloak.yourdomain.com**

###### **admin user:** admin

###### **admin password:** admin

**Swagger UI Spring Cloud Gateway REST API Documentation**, secured with **Keycloak Server** will be available here: **https://bookapi.yourdomain.com**

###### **manager user:** admin

###### **manager password:** admin

###### **regular user:** user

###### **regular password:** user

###### **Oauth2 Client:** book-app


**Grafana Observability Stack**, will be available here: **https://grafana.yourdomain.com**

###### **user:** user

###### **password:** see step-08


### Step 01 - Clone repositories

**https://github.com/greeta-bookshop-01/book-api** (API Source Code and Docker Images Repository)

**https://github.com/greeta-bookshop-01/book-infra** (Terraform Infrastructure and GitOps Pipeline)

### Step-02: Prepare Your AWS Account

- make sure you have AWS Account with enough permissions

- make sure you have your own registered domain and hosted zone

-  create wildcard AWS Certificate for your domain: "*.yourdomain.com" (you will need ssl_certificate_arn later)

### Step-03: Prepare Your Github Account

- make sure you have your own Github Account or Organization

- clone book-api and book-infra repositories to your github profile or organization

- In your cloned book-api Github Repository, go to Settings -> Secrets and Variables -> Actions -> New Repository Secret and create DISPATCH_TOKEN secret with the value of your personal github token (You need to create personal token in Developer Settings and make sure you give it workflow permissions)

- make sure your book-api repository docker images is public by default (you need to change it in github settings: https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)


### Step-04: Prepare API Source Code and Github Actions Workflow:

- go to the root directory of your cloned book-api, book-infra github repository

- Edit "**.github/workflows**" files: replace "**greeta-book-01**" with the name of your github profile or organization; replace "**book-api and book-infra**" with the names of your cloned or forked repositories (or leave the names like this if you don't want to change the names); replace "**master**" with the name of your main branch (or leave it like this, if you don't want to change, but please, note that you would have to change default main branch name in github settings)


### Step-05: Prepare Terraform Infrastructure:

- go to the root directory of your cloned book-infra github repository

- create terraform.auto.tfvars in your book-infra repository and provide your own aws_region and ssl_certificate_arn

```
aws_region = "eu-central-1"
environment = "dev"
business_division = "it"
cluster_name = "book-cluster"
ssl_certificate_arn = "arn:aws:acm:eu-central-1:your-certificate-arn"
```

- replace "greeta.net" in terraform files of book-infra repository, with the name of your domain (please, use search to find all files, where "greeta.net" is used)

- Commit your book-infra changes to github (don't worry, terraform.auto.tfvars is in .gitignore and it won't be committed)
```
git add .
git commit -m "your comment"
git push origin
````

### Step-06: Build Docker Images with Github Actions

- go to the root directory of your cloned book-api github repository

- Commit your book-api changes to github (it should trigger creation of docker images pipeline and also trigger book-infra pipeline)
```
git add .
git commit -m "your comment"
git push origin
````

- wait until book-api pipeline in github is finished and book-infra pipeline is started

- book-infra pipeline automatically changes docker image versions to the versions of docker images, created in book-api pipeline and pushes new docker image versions to book-infra repository


### Step-07: Deploy Infrastructure to AWS with Terraform:

- go to the root directory of your cloned book-infra github repository

- pull changes from books-infra repository and run terraform

```
git pull
terraform apply --auto-approve
```

- if terraform script is failed during creation of grafana observability stack, please, run terraform apply --auto-approve again (it sometimes happens when kubernetes cluster is not ready yet)

- grafana observability stack will be available by url: https://grafana.yourdomain.com; username: user; password: you should see the password in the output of terraform script. Sometimes it is empty. In this case, you can get the password with this command:

```
kubectl get secret --namespace observability-stack loki-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode;
```

### Step-08: Test your Microservices:

- go to "**https://bookapi.yourdomain.com**"

- you should see successfully loaded "**Swagger UI REST API Documentation**" page with drop-down selection of microservices
- Select book or ERP microservice from the drop-down list
- Click **Authorize** button and login with admin/admin (full access) or user/user (limited access)
- In **Authorize** dialog window you should also provide the name of the OAuth2 Client (**book-app** )
- After successfull authorization, try any REST API endpoint
- Go to https://grafana.yourdomain.com and find the logs and traces, generated by the endpoints (Find "Explore" menu, then go to "Loki", select "app" and then select the name of the microservice and then "Run Query")


##### Congratulations! You sucessfully tested Cloud-Native Microservices GitOps Pipeline on AWS with Terraform, Kubernetes, Spring Cloud Gateway and Keycloak!
- ##### Now you can deploy your own docker containers to AWS EKS cluster!
- ###### You also implemented AWS Load Balancer Controller with External DNS and SSL, which act as a Gateway for your microservices and automatically creates sub-domains, using your wildcard AWS Certificate)
- ###### You also implemented Spring Cloud Gateway and Swagger UI REST API Documentation for your microservices, which allows you to select REST API Documentation Microservices from the drop-down list and authorize with Keycloak
- ###### Now you can add any number of microservices to your Kubernetes Cluster and use only one Kubernetes Ingress Load Balancer and Spring Cloud Gateway for all these microservices

- ##### You successfully deployed Keycloak Authorization Server, which protects your Swagger UI REST API Documentation Page
- ##### Spring Security seamlessly handled the entire process of calling the Keycloak OAuth2 Authorization Server to authenticate the user
- ###### Now you can protect any number of microservices by your Keycloak Server and use Single Sign-On Authentication for all these microservices


### Step-09: Clean-Up:

Please make sure you run terraform-destroy.sh script, instead of just calling terraform destroy (otherwise you will have issues with deletion of kubernetes ingress resources by terraform)

```
sh terraform-destroy.sh  
```

If you accidentally used **terraform destroy** directly, then wait for about 20 minutes, then you need to remove all resources manually with AWS Console: EC2 - Load Balancer, EKS - book-cluster, VPC - sandbox, Route 53 Records, IAM Roles. If VPC is failed to delete, wait for EKS cluster to finish deletion.
