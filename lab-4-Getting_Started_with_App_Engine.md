# Course: Google Cloud Platform Fundamentals - Core Infrastructure

## Module: Applications in the Cloud

### Lab: GCP Fundamentals: Getting Started with App Engine

### **Task 1: Initialize App Engine**

- We initialize our project using a preset environment variable provided by Cloud Shell
    > gcloud app create --project=$DEVSHELL_PROJECT_ID

- For this lab we're working with a pre-existing application which we clone
    > git clone https://github.com/GoogleCloudPlatform/python-docs-samples

- Navigate into the project to execute commands
    > cd python-docs-samples/appengine/standard_python3/hello_world

### **Task 2: Run Hello World application locally**

- In order to run the application, a Python environment must be setup. Python virtual environments offer the benefit of isolating package installations from the system.

    > sudo apt-get update

    > sudo apt-get install virtualenv [press Y if prompted]

    > virtualenv -p python3 venv

- Activate the virtual environment
  
    > source venv/bin/activate

- Next, we install application dependencies. Ensure this is run inside the project folder
    > pip install  -r requirements.txt

- Finally we run the application
    > python main.py

### **Task 3: Deploy and run Hello World on App Engine**

- Stop the previous running instance by pressing CTRL + C:

- Ensure we're still in the application directory. If not navigate to it:
    > cd ~/python-docs-samples/appengine/standard_python3/hello_world

- Deploy the web application to App Engine. This app deploy command uses the app.yaml file to identify project configuration.
    > gcloud app deploy

- Test the deployment by opening the application in a new window/tab
    > gcloud app browse

### **Task 4: Disable the application**

This doesn't seem to have a corresponding command so must be done via console
