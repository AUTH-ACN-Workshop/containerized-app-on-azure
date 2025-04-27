# Assignment 1: Build, Push, and Deploy EspoCRM Docker Image on Azure 🏗️🚀

In this assignment, you will build the EspoCRM Docker image, push it to Azure Container Registry (ACR), and deploy it to Azure App Service. This will give you hands-on experience with building and deploying containerized applications in the cloud.

---

## ✅ Step 1: Create an Azure Container Registry (ACR)

An Azure Container Registry (ACR) is a private registry for storing and managing container images. You will create an ACR instance to store your EspoCRM Docker image.

1. In the Azure Portal, search for "Container Registries" in the top search bar.
2. Click on **➕ Create** to create a new ACR instance.
3. Fill in the following details:
   - **Subscription**: Select your Azure subscription.
   - **Resource Group**: Create a new resource group (e.g., `rg-espocrm-lab`).
   - **Registry Name**: Enter a **globally unique** name for your ACR (e.g., `acrespocrm<your-username>`).
   - **Location**: Select a region close to you (e.g., `West Europe`).
   - **Pricing Plan**: Select `Basic`.
4. [Optional] Explore the rest of the settings and leave them as default.
5. Click on **Review + Create** and then **Create** to create the ACR instance.

    > ⌛ Wait for the deployment to complete. This may take a few minutes.
    > [Optionally] Learn more about [Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-intro).

6. Once the deployment is complete, navigate to your ACR instance in the Azure Portal.
7. In the left sidebar, under **Settings**, click on **Access keys**. Here, you will find the `Username` and `Password` for your ACR instance. You will need these credentials to log in to ACR from GitHub Actions.

    > ✏️ **Note**: You might need to enable the `Admin user` option to access the `Username` and `Password` values.

    > 💡 **Tip**: Make sure to copy the `Username` and `Password` values somewhere safe, as you will need them later.

## ✅ Step 2: Build & Push the EspoCRM Docker Image using GitHub Actions

GitHub Actions is a CI/CD tool that allows you to automate your software development workflows. In this case, we'll use it to build and push the EspoCRM Docker image to Azure Container Registry (ACR).

1. In your forked `espocrm-docker` repository, navigate and explore the `apache/Dockerfile` file. This file contains the instructions for building the EspoCRM Docker image.

<!--
Append the following lines to the end of the `apache/Dockerfile` file in case there is a problem with the permissions of the `data` directory.

```dockerfile
# Fix permissions for EspoCRM 'data' directory
RUN cd /var/www/html && \
    mkdir -p data && \
    find data -type d -exec chmod 775 {} + && \
    chown -R www-data:www-data .

```
--->

2. Configure the GitHub Actions workflow to build and push the Docker image to ACR:
   - In your forked repository, navigate to the **Settings** tab.
   - Scroll down to the **Secrets and variables** section in the left sidebar and click on **Actions**.
   - Click on **New repository secret** and add the following secrets:
     - `AZURE_ACR_NAME`: The name of your Azure Container Registry (e.g., `acrespocrm<your-username>`).
     - `AZURE_ACR_USERNAME`: The username for your ACR (usually the same as your ACR name).
     - `AZURE_ACR_PASSWORD`: The password for your ACR (already copied from the ACR instance in Azure Portal earlier).
3. Create a new Action workflow file in your forked repository:
   - Create a new file in the `.github/workflows` directory and name it `build-push-image-acr.yml`.
   - Copy and paste the following code into the file:
        ```yaml
        name: Build and Push EspoCRM Image

        on:
        push:
            branches:
            - master

        jobs:
        build-and-push:
            runs-on: ubuntu-latest

            steps:
            - name: Checkout Code
                uses: actions/checkout@v3

            - name: Log in to ACR
                run: echo "${{ secrets.AZURE_ACR_PASSWORD }}" | docker login ${{ secrets.AZURE_ACR_NAME }}.azurecr.io -u ${{ secrets.AZURE_ACR_USERNAME }} --password-stdin

            - name: Build Docker Image
                run: docker build -t ${{ secrets.AZURE_ACR_NAME }}.azurecr.io/espocrm:latest ./apache

            - name: Push Docker Image
                run: docker push ${{ secrets.AZURE_ACR_NAME }}.azurecr.io/espocrm:latest
        ```
    - Commit and push the changes to your forked repository. This will trigger the GitHub Actions workflow to build and push the Docker image to ACR.
4. Monitor the progress of the GitHub Actions workflow by navigating to the **Actions** tab in your forked repository. Click on the latest workflow run to see the logs and status of each step.

    > ⌛ Wait for the workflow to complete. This may take a few minutes.

5. Once the workflow is complete, navigate to your ACR instance in the Azure Portal.
6. In the left sidebar, under **Settings**, click on **Repositories**. You should see a new repository named `espocrm` with the latest image tagged as `latest`.

## ✅ Step 3: Create an Azure Database for MySQL

EspoCRM requires a MySQL database to store its data. You will create an `Azure Database for MySQL flexible server` instance to use as the database for EspoCRM.

1. In the Azure Portal, search for "Azure Database for MySQL flexible server" in the top search bar.
2. Click on **➕ Create** to create a new MySQL flexible server instance.
3. Select the **Advanced Create** option under **Flexible server**.
4. On `Basics` tab, select the following options:
   - **Subscription**: Select your Azure subscription.
   - **Resource Group**: Select the same resource group you created earlier (e.g., `rg-espocrm-lab`).
   - **Server Name**: Enter a **globally unique** name for your MySQL server (e.g., `mysql-espocrm<your-username>`).
   - **Region**: Select a region close to you (e.g., `West Europe`).
   - **Authentication Type**: Select `MySQL authentication only`.
     - **Administrator Login**: Enter a username (e.g., `dbadmin`).
     - **Password**: Enter a strong password (e.g., `StrongPassword123!`).
5. On the `Networking` tab, select the following options:
   - **Public access**: Select `Allow public access to this resource through the internet using a public IP address`.
   - Under **Firewall rules**, select `Allow public access from any Azure service within Azure to this server`.
6. [Optional] Explore the rest of the settings and leave them as default.
7. Click on **Review + Create** and then **Create** to create the MySQL flexible server instance.

    > ⌛ Wait for the deployment to complete. This may take a few minutes.
    > [Optionally] Learn more about [Azure Database for MySQL](https://learn.microsoft.com/en-us/azure/mysql/flexible-server/overview).

8. Once the deployment is complete, navigate to your MySQL flexible server instance in the Azure Portal.
9. In the left sidebar, under **Server parameters**, search for `require_secure_transport`, set it to `OFF` and click on **Save**. This will allow EspoCRM to connect to the MySQL server without SSL.

    > ⚠️ **Warning**: Disabling SSL is not recommended for production environments. Use this setting only for testing purposes.
10. In the left sidebar, under **Databases**, click on **➕ Add** to create a new database.
    - **Database name**: Enter a name for your database (e.g., `espocrm`).
    - Click on **Save** to create the database.

## ✅ Step 4: Create an Azure App Service

Azure App Service is a fully managed platform for building, deploying, and scaling web apps. It supports deploying containerized applications or code in various languages and frameworks. You will create an App Service instance to host the EspoCRM application, using the Docker image you built and pushed to ACR.

1. In the Azure Portal, search for "App Services" in the top search bar.
2. Click on **➕ Create** (Web App) to create a new App Service instance.
3. On `Basics` tab, select the following options:
   - **Subscription**: Select your Azure subscription.
   - **Resource Group**: Select the same resource group you created earlier (e.g., `rg-espocrm-lab`).
   - **Name**: Enter a **globally unique** name for your App Service (e.g., `app-espocrm-<your-username>`).
   - **Publish**: Select `Container`.
   - **Operating System**: Select `Linux`.
   - **Region**: Select a region close to you (e.g., `West Europe`).
   - **Linux Plan**: Select **Create new** and choose the following options:
     - **Name**: Enter a name for your App Service plan (e.g., `asp-espocrm`).
     - **Pricing Plan**: Select `Basic B1`.
4. On the `Container` tab, select the following options:
   - **Image Source**: Select `Azure Container Registry`.
   - **Registry**: Select your ACR instance (e.g., `acrespocrm<your-username>`).
   - **Authentication**: Select `Admin credentials`.
   - **Image***: Enter `espocrm`.
   - **Tag**: Enter `latest`.
5. [Optional] Explore the rest of the settings and leave them as default.
6. Click on **Review + Create** and then **Create** to create the App Service instance.

    > ⌛ Wait for the deployment to complete. This may take a few minutes.
    > [Optionally] Learn more about [Azure App Service](https://learn.microsoft.com/en-us/azure/app-service/overview).

7. Once the deployment is complete, navigate to your App Service instance in the Azure Portal.
8. Navigate to the `Environment Variables` section in the left sidebar and add the following environment variables:
   - `ESPOCRM_DATABASE_HOST`: The hostname of your MySQL flexible server instance (e.g., `mysql-espocrm<your-username>.mysql.database.azure.com`). This can be found in the `Overview` section of your MySQL flexible server instance.
   - `ESPOCRM_DATABASE_NAME`: The name of the SQL database you created earlier (e.g., `espocrm`).
   - `ESPOCRM_DATABASE_PASSWORD`: The password you set for the MySQL flexible server instance (e.g., `StrongPassword123!`).
   - `ESPOCRM_DATABASE_USER`: The username you set for the MySQL flexible server instance (e.g., `dbadmin`).
   - `ESPOCRM_SITE_URL`: The URL of your App Service instance (e.g., `https://app-espocrm<your-username>.azurewebsites.net`). This can be found in the `Overview` section of your App Service instance.
  
   These environment variables will be used by EspoCRM application to connect to the MySQL database and configure the application settings.
9. Click on **Apply** to save the changes.
10. In the left sidebar, click on **Deployment Center**. From here, on the `Logs` tab, you can monitor the deployment process of the docker image to the App Service instance. You can also view the logs of the application by clicking on the `Log stream` tab.

    > ⌛ Wait for the deployment to complete. This may take a few minutes.

11. Once the deployment is complete, navigate to the `Overview` section of your App Service instance and click on the URL to access your EspoCRM application.

Congratulations! You have successfully built, pushed, and deployed the EspoCRM Docker image on Azure! 🏆🏆

✏️ **Note**: The default username and password for EspoCRM are `admin` and `password`, respectively. Once you logged in, navigate to the web application and explore the features of EspoCRM.

## ⚠️ Important: Clean Up Resources

After completing the assignments, remember to clean up your Azure resources to avoid unnecessary charges upon your free credits. You can delete the resource group you created for this assignment, which will remove all resources within it.

1. In the Azure portal, navigate to **Resource groups**.
2. Select the resource group you created for this assignment (e.g., `rg-hello-cloud-devops`).
3. Click on **Delete resource group**.