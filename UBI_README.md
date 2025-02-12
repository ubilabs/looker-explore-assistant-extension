# UBI Setup Guide

This guide walks you through setting up the backend for storing prompt examples in BigQuery and deploying the Explore Assistant extension. Follow each step carefully.

> For more in depth explanations on each of the steps, read the readme files in the directories respectively

1. Create a Google Cloud Project

   - Objective: Set up a new Google Cloud project dedicated to your backend services and BigQuery storage.
   - Steps: In the Google Cloud Console, create a new project.

2. Deploy the BigQuery Backend with Terraform

   - Prerequisite: Ensure Terraform is installed on your machine.
   - Steps:

     - Open your terminal and navigate to the backend folder:

     ```bash
     cd explore-assistant-backend
     ```

     To deploy the BigQuery backend (Prototype):

     ```bash
     cd terraform

     export TF_VAR_project_id=(PASTE BQ PROJECT ID HERE)
     export TF_VAR_use_bigquery_backend=1
     export TF_VAR_use_cloud_function_backend=0

     terraform init
     terraform plan
     terraform apply
     ```

   > Note: These commands use your default gcloud application credentials.

3. Configure Google Cloud Permissions

   > The previous step will have created BQ resources and also added a bq service account.

   - Objective: Grant the BigQuery service account the necessary permissions to use the model.(`bigquery.connections.use` permission)
   - Steps: In the Google Cloud Console, update your project’s IAM settings to assign the required roles to the BigQuery service account.

4. Set Up a New Looker Project

   - Prerequisite: Create a new GitHub repository for the Looker project.
   - Steps:
     • Connect the repository to Looker.
     • Configure a database connection in Looker to access the table created in step 2.

5. Prepare the Explore Assistant Extension

   - Prerequisite: Ensure that the BigQuery dataset contains example prompts.
   - Steps:
     • Navigate to the explore-assistant-extension directory.
     • Fill out the environment file with your project details, including the model ID from inside the BigQuery dataset.
     • Install the project dependencies by running: npm install
     • In your Looker instance, switch to development mode.
     • Upload the manifest.lkml file to your Looker project.
     • Start the extension in development mode by running: npm run start
     - Note: The development server is not accessible via localhost:8080; access it via the “Applications/Explore Assistant” section in Looker.
       • When ready for production, build the extension by running: npm run build
     - After building, upload the generated bundle.js file to your Looker project.
     - Edit the manifest.lkml file to remove (or comment out) the URL field, leaving only the file field pointing to bundle.js.

6. Commit and Deploy Your Changes

   - Objective: Finalize and deploy your extension.
   - Steps: Commit your changes to GitHub and push them to your production branch.

7. Generate Example Prompts

   > You can either run this notebook in a google colab context or use it inside VSCode. When running in VSCode, select the virtual env kernel as the python interpreter. If default credential authentication errors arise a solution may be to restart VSCode/the python kernel (Though you may want to check the users project permissions to use the enabled Vertex AI API first).

   - Prerequisite:

     - Use the provided Jupyter Notebook in the explore-assistant-training folder.
     - User api credentials (client/ ID & secret)
       - Visit looker `Admin/Users/<Your_User>/` and click on "Edit Keys"
     - The enabled Vertex AI API

   - Steps:
     - Fill in fields of `base.looker.ini` file and rename it to `looker.ini`
     - Create a python venv if not done already and activate it:
       > Best to run this command in the root dir of this project to use the virtual env for the whole project.
       ```bash
       python3 -m venv ./venv
       . ./venv/bin/activate
       ```
     - Generate new prompt examples for training the language model using notebook.
       - You must update the explore view variables at some poits inside the notebook
     - **Review the generated `examples.json`** and delete lines that are not in JSON format or duplicates!
       - You may want to run this a couple times and see different results.
     - Replace the examples JSON file in the `explore-assistant-examples` directory with the newly genrated ones

8. Upload Training Examples to BigQuery

   - Objective: Update your BigQuery dataset with the new examples.
   - Steps:
     - Navigate to the `explore-assistant-examples` directory
     - Modify the env file. The google project id for the looker connection, the dataset id (like "explore-assistant"), and the looker explore id (like "ubilabs-data-warehouse:time_entry")
     - Use the scripts in the explore-assistant-examples directory to upload or append your training examples to the BigQuery tables.
       - Check the variables in the script respectively to see if the json file name actually points to the relevant json file

   > The `examples.json` examples are use for the actual chat improvements and the `samples.json` file is used for the recommended queries when opening a new chat in the explore assistant.

For further details, consult the following documentation:

- [Backend Docs](explore-assistant-backend/README.md)
- [Frontend Docs](explore-assistant-extension/README.md)
- Documentation in the [explore-assistant-training notebook](explore-assistant-training/looker_explore_assistant_training.ipynb) and [explore-assistant-examples](explore-assistant-examples/README.md) directories.
