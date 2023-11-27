# Web Application Deployment Automation with Python

In this project we are going to create a React web application using Python and automatically starts the web server on port 3000. The task would invove the following:
- Setting up the neccessary environment.
- Installing Dependencies
- Cloning github repository
- starting the web application

We are going to run the application on a vagrant Virtual machine running Ubuntu 20.04

## Step 1:

In the home directory of the VM create a folder `script`, change directory to script folder and create a file `create_webapp.py` and add the following python code in create_webapp.py.

```bash
mkdir script
cd script
sudo vi create_webapp.py
```

```python
import subprocess
import os
import json

project_name = "mywebapp"

def run_command(command):
    try:
        subprocess.run(command, shell=True, check=True)
    except subprocess.CalledProcessError as e:
        print(f"Error running the command: {e}")
        exit(1)

def update_json_file():
    # Specify the path to your package.json file
    package_json_path = "/home/vagrant/script/mywebapp/mern-test/package.json"

    # Define the new "dev" script value
    new_dev_script = "webpack-dev-server --open --host 0.0.0.0 --port 3000"
    
    # Read the package.json file
    with open(package_json_path, "r") as package_json_file:
        package_json_data = json.load(package_json_file)
    
    # Modify the "dev" script value
    package_json_data["scripts"]["dev"] = new_dev_script
    print("checking package_json_data...")
    print(package_json_data)

    # Write the updated JSON object back to the package.json file

    with open(package_json_path, "w") as package_json_file:
        json.dump(package_json_data, package_json_file, indent=2)
        print("checking package_json_data")
        print(package_json_data)
    print("Updated the 'dev' script in package.json")

def create_webapp():
    try:
        print(f"Creating directory{project_name}")
        run_command(f"mkdir -p {project_name}")
        print(f"navigating to {project_name}")
        os.chdir(project_name)
        print("check directory... ")
        run_command(f"pwd")
        print("Installing neccessary software... ")
        print("running command sudo apt-get update etc... ")
        run_command("sudo apt-get update")
        run_command("sudo apt-get install -y nodejs npm")

        print("Installing git... ")
        run_command("sudo apt install git")
        print("checking git version... ")
        run_command("git --version")

        print("Cloning application repository... ")
        print("running command git clone... ")
        # Clone your web application repository (replace with your repo URL)
        run_command(f"git clone https://github.com/thinkC/mern-test.git")
        print("check directory... ")
        run_command(f"pwd")
        os.chdir("mern-test")
        print("check directory... ")
        run_command(f"pwd")
        run_command("sudo npm install")

        #Call update json file function
        update_json_file()

        # Start the React app on port 3000
        print("Starting react server... ")
        run_command("npm run dev")
    except subprocess.CalledProcessError as e:
        print(f"Error running a command: {e}")
        exit(1)

if __name__ == "__main__":
    create_webapp()


```

Note: The Function update_json_file() updated the package.json file below

from

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start-dev": "concurrently \"npm run server-dev\" \"npm run dev\"",
    "start": "webpack",
    "dev": "webpack-dev-server 
  },
```
To below. This would allow us to browse the application on our HOST PC

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start-dev": "concurrently \"npm run server-dev\" \"npm run dev\"",
    "start": "webpack",
    "dev": "webpack-dev-server --open --host 0.0.0.0 --port 3000"
  },

```
## Step 3:

The script also automatically starts the server and open the browser on `localhost:3000`or you can manually browse the application by browsing `localhost:3000`

![homepage](https://github.com/thinkC/new-devops-projects/blob/master/parent-img/webapp-python/1.img.png?raw=true)

Note:
In the `subprocess.run` function, the `shell` and `check` parameters are used to control how the subprocess is executed and how errors are handled.

1. `shell=True`:

   - When `shell` is set to `True`, it tells `subprocess.run` to run the command through the system shell.

2. `check=True`:
   - When `check` is set to `True`, it instructs `subprocess.run` to raise a `CalledProcessError` exception if the executed command returns a non-zero exit status.


## Conclusion:

We are ale to create a web application by with Python script.