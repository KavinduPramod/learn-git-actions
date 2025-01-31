# Test Project to Learn How to Deploy a React App to a VM
We will create a simple ReactJS project and deploy it on a virtual machine, then automate the continuous deployment with GitHub Actions. The main target is to understand GitHub Actions and set up a basic CI/CD pipeline.

## Prerequisites
#### For Development

 - Node.js
 - Typical development environment
 
#### For Deployment
 - A GitHub repository
 - A VM (Digital Ocean droplet in this case)
 
## Steps
 - Create a React App with Vite

Since Vite provides a fast build process, we will use it to create our React project. Open your command prompt/terminal and navigate to your preferred folder location for this project. In this case, I am using cmd in Windows 10. I'll create a folder named git-actions-test on the desktop and navigate inside.

```
cd desktop
mkdir git-actions-test
cd git-actions-test
```

Next, let's create a Vite + React project.

```
npm create vite@latest
```
Give it a name you prefer; I went with learn-git-actions. Navigate inside the created Vite project and open it in VS Code or any editor you prefer. I'll use VS Code.
```
cd learn-git-actions
code .
```
To keep things simple, we will use the terminal in VS Code. Close the cmd and press `Ctrl + ~`  to open the terminal. Keep the terminal open and let's develop a very simple React project. Start by editing `src/App.jsx`.

Install npm and run the project in the development environment.
```
npm install
npm run dev
```
Change the project as you like. I'll create a simple to-do list to add and delete tasks. You can copy the code from the Git URL .

> https://github.com/KavinduPramod/learn-git-actions.git

Once the changes are made, let's initialize this as a Git repository and commit the changes.
```
git init
git add .
git commit -m "initial commit"
```
The git init command creates an empty repository where your terminal is, git add . stages the changes, and the other command commits them.

Now, let's create a GitHub repository and copy the link. We can link that remote repository to the local repository where we just made commits. Once the repository is created in GitHub, copy the repository URL and come back to the VS Code terminal. In my case, I created the repository named learn-git-actions. Now, in the VS Code terminal, type:
```
git remote add origin <your git hub repo URL>
```
Now we can push the committed changes to the remote repository. Note: When creating the remote repo, make sure not to create any branches. If you did, change the below command to git push -u <name of the remote branch you created (e.g., main).
```
git push -u origin master
```
&nbsp;
 - Set Up GitHub Actions

Now, make sure our local and remote repositories are synced and can push and pull changes. If everything is done correctly, we will have a remote and local repository with one single Git branch.

If you look at the Actions tab in the GitHub repository, you will see a bunch of configurations under deployment, automation, and CI/CD. Try adding these configurations and see what happens. Whatever these configurations are, they will add a .yaml file to the repository. This .yaml file needs to be in the `.github/workflows/` folder.

Anyhow, we can take matters into our own hands and create the configuration file ourselves. Let's open the local repository (VS Code) and create a `.github` folder, then create a `workflows` folder inside the `.github` folder, and then create the `.yaml` file with a preferred name. I will create the `.yaml` file as `deploy.yaml` in the VS Code terminal.
```
mkdir .github
cd .github
mkdir workflows
cd workflows
New-Item -Path . -Name "deploy.yml" -ItemType "file"
```

Or just simply use the GUI. However, make sure the `deploy.ym`l file is inside the `.github/workflows/` folder.

Now we need some knowledge about the .`yml` syntax. Below you can see the basics we need, but it is recommended to dive deeper into the documentation and tutorials.
```
name: Defines the workflow name. on: Specifies when the workflow should run (e.g., on push or pull_request). 
jobs: A collection of tasks (each runs on a separate virtual machine).
runs-on: Defines the OS for the job (e.g., ubuntu-latest). 
steps: List of commands executed in order. 
uses: Calls pre-built GitHub Actions (e.g., actions/checkout@v3 to clone the repo). 
run: Runs a shell command in the GitHub-hosted runner. 
with: Passes inputs/parameters to an action. 
${{ secrets.VARIABLE_NAME }}: Securely references GitHub Secrets (e.g., SSH key, IP).
```

Get started with the below links:

[GitHub Actions Documentation](https://docs.github.com/en/actions)
[YouTube Tutorial](https://youtu.be/47zYGHwXPmE?si=DgSZ03mP8CHE7VQO)

Now let's get back to our project. In the deploy.yml file, I have put the below content with comments so we all have a basic idea.

```
name: Build and Deploy React App # The name of the workflow (visible in GitHub Actions UI)

on: # The event(s) that trigger the workflow
  push:
    branches:
      - master # The branch to trigger the workflow on
  pull_request:
    branches:
      - master # The branch to trigger the workflow on

jobs: # The jobs to run
  build: # The name of the job
    runs-on: ubuntu-latest # The OS to run the job on

    steps:
      - name: Checkout # The name of the step
        uses: actions/checkout@v3 # The action to use

      - name: Setup Node.js # The name of the step
        uses: actions/setup-node@v2 # The action to use
        with:
          node-version: '18' # The version of Node.js to use

      - name: Install dependencies # The name of the step
        run: npm install # The command to run

      - name: Build # The name of the step
        run: npm run build # The command to run
```
Let's commit these changes to the remote repository and navigate to the Actions tab. You will see all those jobs we specified in the deploy.yml file are running or completed, or you might have to fix the errors if they occur.
&nbsp;

 - Set Up the VM

Now we need to set up the VM first, then apply deploy jobs into the deploy.yml. For this part, I'm using a Digital Ocean droplet with my PC's SSH registered in the droplet. It is also recommended to create an SSH user in the VM (you will see I am the droplet user under the "dev" username) so the root directory does not get involved.

You might have to create a droplet in Digital Ocean with your PC's SSH access. We will use CLI instead of the Digital Ocean dashboard. So, get ready your droplet IP with your PC's SSH access.


Open a new cmd and type the below to SSH into the droplet.

```
ssh dev@<droplet IP>
```

Obviously, I cannot share the droplet IP because of security concerns. Note that dev@ is the username for the SSH user; otherwise, you can use root@. You will be prompted with some (yes/no/fingerprint) questions if it's the first time; if so, type yes and hit Enter.

Once you are in the droplet, you will be inside /home/dev. This /dev will be the username. You can also find out the current path and the contents of the current path with the below commands:

```
pwd
ls -a
```

You might not see any contents. But we will copy the built React project here and configure Nginx. Nginx is open-source web server software we can use to serve our built React web content. First, let's install Nginx. Run the below commands in the droplet.
```
sudo apt update
sudo apt-get install -y nginx
nginx -v
```
With nginx -v, you will get some output with the version number of Nginx, so you can make sure Nginx is installed. Then we have to check the status, enable, and start Nginx. We can also set Nginx to run at startup with the following commands:
```
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

You will see an active (running) message, so you can make sure Nginx is running. Now we need to copy the React built folder to the droplet. To do this, go back to the VS Code terminal and make sure you are in the root folder, then follow the below commands in the VS Code terminal:
```
npm run build
```
This will create a dist folder inside the project, and this is the folder we need to copy to the droplet. To do that, use the below command:
```
scp -r dist dev@<droplet IP>:/home/dev
```

This -r is used to copy the entire folder, and dev is the username of the droplet user. Now go back to the cmd and run ls -a to make sure you have that dist folder inside `/home/dev/` (or your droplet username).

Now, if you use the `ls -a` command, you will see the dist folder in `/home/dev/`. Now we need to move this dist folder into the `/var/www/ folder`. `/var/www`/ is the default web root directory in Linux (used by Apache & Nginx). This directory has proper security and access controls for serving public files. We also need to give Nginx ownership and set the correct permissions (read & execute). To do all these things, follow the below commands:

```
sudo mv /home/dev/dist /var/www/react-app
sudo chown -R www-data:www-data /var/www/react-app
sudo chmod -R 755 /var/www/react-app
```
Now we can write an Nginx configuration to serve our React application in `/var/www/`. Notice that I have moved the dist folder inside `/var/www/react-app`. Now we will edit the default Nginx config to open the configuration in the nano editor using the below command:
```
sudo nano /etc/nginx/sites-available/default
```

Inside the configuration file, you will see something similar to the below config. CAREFULLY replace the content with the below provided config code:

```
server {
    listen 80; # Listen for HTTP requests on port 80
    server_name _; # Make sure to replace this line with the domain name if you have one; otherwise, remove it

    root /var/www/react-app/dist; # Serve files from this directory
    index index.html; # Load index.html when accessing the root URL

    location / {
        try_files $uri /index.html; # React routing: If file not found, load index.html
    }

    error_page 404 /index.html; # Handle 404 errors by showing React's index.html
}
```

Now, if you open your web browser and navigate to `http://<YOUR_DROPLET_IP>`, you will see our React application working! As for the final touch in the droplet, make sure your SSH is working with the droplet (this has to work since we had SSH access if you created the droplet). Use the below command in a new cmd and see if you can log into the droplet:
```
ssh -i ~/.ssh/id_rsa dev@<DROPLET_IP>
```

Before we close the droplet, we need to allow the dev (or your droplet) user to run systemctl restart nginx without a password because we are going to need to run that using the .yml. To do that:
```
sudo visudo
```

Add the below line at the end:
```
dev ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```

Finally, run the below commands to make sure the user has the correct permissions:
```
sudo chown -R dev:www-data /var/www/react-app
sudo chmod -R 775 /var/www/react-app
```

Now, if you have correctly logged in, you know your SSH is working with the droplet. Now let's deal with GitHub Actions and automate our process. First of all, navigate to the `C:\Users\username\.ssh\id_rsa` (this will be the default path of your SSH keys) directory or `/custom/path/of/.ssh`. In the directory, you will see `id_rsa` and `id_rsa.pub`. Open the `id_rsa` with Notepad or any text editor and copy the entire content. 

**CAUTION**: Do not share the copied content with anyone else.


Now go to GitHub and select our remote repository, then navigate to `Settings` and navigate to `Secrets and variables -> Actions`. You will see a button to add a new repository secret. First, we are going to add the private key we have copied, so add the first secret. Make sure to name the secret as `PRIVATE_SSH_KEY` and in the secret text area, paste your private SSH key.

Then, as the second secret, add your droplet IP address and let's name that secret `DROPLET_HOST`. 

Let's add the droplet username (dev in this case) as the final secret. Let's name the secret `DROPLET_USER` and enter your SSH username for the droplet.

Now that we have everything we need, let's update our `deploy.yml` to see if we can connect to the droplet first! So add the below lines to the `deploy.ym`l in the local repository:
```
- name: Setup SSH Key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.PRIVATE_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.DROPLET_HOST }} >> ~/.ssh/known_hosts
```
This workflow step sets up SSH key authentication by creating an SSH directory, adding a private key, setting appropriate permissions, and adding the droplet host to the known hosts file.

Then finally, we can add the below steps:

```
- name: Deploy Build to Droplet
        uses: appleboy/scp-action@v0.1.4
        with:
          host: ${{ secrets.DROPLET_HOST }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.PRIVATE_SSH_KEY }}
          source: "dist/*"
          target: "/var/www/react-app"

      - name: Restart Nginx
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.DROPLET_HOST }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.PRIVATE_SSH_KEY }}
          script: sudo systemctl restart nginx
  ```
  
These steps deploy a build to a droplet by securely copying files to a specified directory and then restart the Nginx service to apply the changes.

The final `deploy.yml` file will look like this:

```
name: Build and Deploy React App # Name of the workflow (visible in GitHub Actions UI)

on: # Trigger events
  push:
    branches:
      - master # Runs on push to master
  pull_request:
    branches:
      - master # Runs on pull request to master

jobs:
  build: # Job name
    runs-on: ubuntu-latest # OS for the job

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18' # Node.js version

      - name: Install Dependencies
        run: npm install

      - name: Build React App
        run: npm run build

      - name: Setup SSH Key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.PRIVATE_SSH_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H ${{ secrets.DROPLET_HOST }} >> ~/.ssh/known_hosts

      - name: Deploy Build to Droplet
        uses: appleboy/scp-action@v0.1.4
        with:
          host: ${{ secrets.DROPLET_HOST }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.PRIVATE_SSH_KEY }}
          source: "dist/*"  # Copies only contents, not the dist folder itself
          target: "/var/www/react-app"

      - name: Restart Nginx
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.DROPLET_HOST }}
          username: ${{ secrets.DROPLET_USER }}
          key: ${{ secrets.PRIVATE_SSH_KEY }}
          script: sudo systemctl restart nginx
   ```
   
Note: I have added a new trigger event to trigger this workflow on a pull request.

Now, whenever we push changes or make a pull request to the master branch, this will build the React app and publish it on the droplet. Now make a simple change in the to-do app and check.