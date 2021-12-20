# Steps to build infrastructure as code with Terraform Cloud Build

## Configuring your **dev** environment

1. In the Google Cloud Console, on the project selector page, select or create a Google Cloud project.

2. Make sure that billing is enabled for your Cloud project.

3. In the Cloud Console, activate Cloud Shell.

4. In Cloud Shell, get the ID of the project you just selected:

```gcloud config get-value project 
```

5. We need the Project ID in some the commands, we need to make it available. We need to export the Project ID.

```export PROJECT_ID='project_id'
```
6. Enable the required APIs:

```gcloud services enable cloudbuild.googleapis.com compute.googleapis.com
```
7. If you've never used Git in Cloud Shell, configure it with your name and email address:

```git config --global user.email "your-email-address"
git config --global user.name "your-name"
```


## Setting up your GitHub repository
8. To get started, you fork the ishgem007/gcp-pipeline repository.

9. In Cloud Shell, clone this forked repository, replacing your-github-username with your GitHub username:

```cd ~
git clone https://github.com/your-github-username/gcp-pipeline.git
cd ~/solutions-terraform-cloudbuild-gitops
```

The code in this repository is structured as follows:

- The ```environments/``` folder contains subfolders that represent environments, such as dev and prod

- The ```modules/``` folder contains inline Terraform modules. These modules represent logical groupings of related resources and are used to share code across different environments.

- The cloudbuild.yaml file is a build configuration file that contains instructions for Cloud Build, such as how to perform tasks based on a set of steps

For dev and prod environments, the following steps are executed:

    1. terraform init
    2. terraform plan
    3. terraform apply

## Configuring Terraform to store state in a Cloud Storage bucket

In the following steps, you create a Cloud Storage bucket and change a few files to point to your new bucket and your Google Cloud project.

1. In Cloud Shell, create the Cloud Storage bucket:

```PROJECT_ID=$(gcloud config get-value project)
gsutil mb gs://${PROJECT_ID}-tfstate
```

Pleae note: You may have issue with this if ```export PROJECT_ID='project id' ``` is not executed prior to this.

2. In Cloud Shell, create the Cloud Storage bucket:
```gsutil versioning set on gs://${PROJECT_ID}-tfstate
````

3. Replace the PROJECT_ID placeholder with the project ID in both the terraform.tfvars and backend.tf files:

```cd ~/gcp-pipeline
sed -i s/PROJECT_ID/$PROJECT_ID/g environments/*/terraform.tfvars
sed -i s/PROJECT_ID/$PROJECT_ID/g environments/*/backend.tf
```

4. Check whether all files were updated:

```git status
```

5. Commit and push your changes:

```git add --all
git commit -m "Update project IDs and buckets"
git push origin 
```

## Granting permissions to your Cloud Build service account

To allow Cloud Build service account to run Terraform scripts with the goal of managing Google Cloud resources, you need to grant it appropriate access to your project. 

1. In Cloud Shell, retrieve the email for your project's Cloud Build service account:

```CLOUDBUILD_SA="$(gcloud projects describe $PROJECT_ID \
    --format 'value(projectNumber)')@cloudbuild.gserviceaccount.com"```

2. Grant the required access to your Cloud Build service account:
```gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member serviceAccount:$CLOUDBUILD_SA --role roles/editor```

## Directly connecting Cloud Build to your GitHub repository
This installation allows you to connect your GitHub repository with your Google Cloud project so that Cloud Build can automatically apply your Terraform manifests each time you create a new branch or push code to GitHub.

The following steps provide instructions for installing the app only for the gcp-pipeline repository, but you can choose to install the app for more or all your repositories

1. Go to the GitHub Marketplace page for the Cloud Build app:
https://github.com/marketplace/google-cloud-build

- If this is your first time configuring an app in GitHub: Click Setup with Google Cloud Build at the bottom of the page. Then click Grant this app access to your GitHub account.

- If this is not the first time configuring an app in GitHub: Click Configure access. The Applications page of your personal account opens.

2. Click Configure in the Cloud Build row.

3. Select Only select repositories, then select solutions-terraform-cloudbuild-gitops to connect to the repository.

4. Click Save or Install—the button label changes depending on your workflow. You are redirected to Google Cloud to continue the installation.

5. Sign in with your Google Cloud account. If requested, authorize Cloud Build integration with GitHub.

6. On the Cloud Build page, select your project. A wizard appears.

7. In the Select repository section, select your GitHub account and the solutions-terraform-cloudbuild-gitops repository.

8. If you agree with the terms and conditions, select the checkbox, then click Connect.

9. In the Create a trigger section, click Create a trigger and add the trigger name.

10. In the Event section, select Push to a branch, enter .* in the Base Branch field, and then click Create.

The Cloud Build GitHub app is now configured, and your GitHub repository is linked to your Google Cloud project. From now on, changes to the GitHub repository trigger Cloud Build executions, which report the results back to GitHub.




## Just for demostration, this step will:
 1. Configure an apache2 http server on network '**dev**' and subnet '**dev**-subnet-01'
 2. Open port 80 on firewall for this http server 


```bash
cd ../environments/dev
terraform init
terraform plan
terraform apply
terraform destroy
```

## Promoting the environment to **production**

Once you have tested your app (in this example an apache2 http server), you can promote your configuration to prodution. This step will:
 1. Configure an apache2 http server on network '**prod**' and subnet '**prod**-subnet-01'
 2. Open port 80 on firewall for this http server 

```bash
cd ../prod
terraform init
terraform plan
terraform apply
terraform destroy
```
