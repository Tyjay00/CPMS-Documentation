# Cloud Security Posture Assessment (CSPM)

This document provides a step-by-step guide on how to perform a Cloud Security Posture Assessment (CSPM) using ScoutSuite on an Ubuntu Virtual Machine, connecting to a AWS account via AWS IAM Identity Center (SSO).


## 🔗 Project Website  
Check out the live demo here: [CSPM] (https://cspm.tyrones.codes/)

---
**Goal:** Identify misconfigurations, compliance violations, and security risks in your cloud environment.

**Tool Used:** ScoutSuite (Open-source CSPM tool)

**Authentication Method:** AWS IAM Identity Center (SSO)

**Environment:** Ubuntu Desktop Virtual Machine (using VirtualBox)

---

## 1. Setting up the Ubuntu Virtual Machine

This section covers installing VirtualBox and setting up an Ubuntu Desktop VM, which will host our security tools.

### 1.1. Download and Install VirtualBox

1.  **Download VirtualBox:**
    * Go to the official VirtualBox Downloads page: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
    * Download the appropriate installer for your host operating system (Windows, macOS, Linux).

2.  **Install VirtualBox:**
    * Run the downloaded installer. Follow the on-screen prompts, typically accepting default options.

### 1.2. Download Ubuntu Desktop ISO

1.  **Download Ubuntu Desktop ISO:**
    * Visit the official Ubuntu Desktop download page: [https://ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)
    * Download the latest LTS (Long Term Support) version (e.g., Ubuntu 22.04 LTS). This will be a large `.iso` file.

### 1.3. Create a New Virtual Machine in VirtualBox

1.  **Open VirtualBox Manager.**

2.  Click the **"New"** button to start the Create Virtual Machine wizard.

3.  **Name and Operating System:**
    * **Name:** Enter `Ubuntu-ScoutSuite-Demo`.
    * **ISO Image:** Browse and select the Ubuntu `.iso` file you downloaded.
    * Check **"Skip Unattended Installation"**.
    * Click **"Next"**.

4.  **Hardware (Base Memory and Processors):**
    * **Base Memory (RAM):** Set to `4096 MB` (4 GB) or more (within the green range).
    * **Processors:** Set to `2` CPUs or more (within the green range).
    * Click **"Next"**.

5.  **Virtual Hard Disk:**
    * **Disk Size:** Allocate at least `25 GB`. `Dynamically allocated` is suitable for demos.
    * Click **"Next"**.

6.  **Summary:** Review settings and click **"Finish"**.

### 1.4. Install Ubuntu on the Virtual Machine

1.  **Start the VM:** In VirtualBox Manager, select your `Ubuntu-ScoutSuite-Demo` VM and click **"Start"**.

2.  **Follow Ubuntu Installation Prompts:**
    * Select **"Install Ubuntu"**.
    * Choose your **Language** and **Keyboard Layout**.
    * Select **"Normal installation"** and **"Download updates while installing Ubuntu"**.
    * For "Installation type", choose **"Erase disk and install Ubuntu"** (this only affects the virtual disk).
    * Set your **Time Zone** (e.g., Africa/Johannesburg).
    * Create your **user account** (your name, username, and a strong password). **Remember these credentials!**

3.  **Complete Installation:** Wait for the installation to finish, then click **"Restart Now"**.

4.  **Log In:** Log into your new Ubuntu Desktop VM using the username and password you created.

### 1.5. Install VirtualBox Guest Additions (Highly Recommended)

1.  From the VirtualBox menu bar (at the top of your VM window), go to **Devices > Insert Guest Additions CD Image...**

2.  In Ubuntu, a pop-up might ask to run the software. Click **"Run"** and enter your password.
    * If it doesn't auto-run, open the file manager, navigate to the "VBox_GAs_..." CD drive, right-click in an empty space, select "Open in Terminal", and run: sudo sh VBoxLinuxAdditions.run

3.  After installation, **restart your Ubuntu VM** (sudo reboot).

---

## 2. Installing ScoutSuite and Preparing AWS CLI for SSO

This section details how to install the necessary tools in your Ubuntu VM.

### 2.1. Open Terminal and Create Python Virtual Environment

1.  Open a Terminal in your Ubuntu VM (Ctrl + Alt + T).

2.  **Update Package Lists:**
    ```bash
    sudo apt update
    ```

3.  **Install `python3-venv` (if not already installed):**
    ```bash
    sudo apt install python3-venv -y
    ```

4.  **Create a Python Virtual Environment:**
    ```bash
    python3 -m venv ~/scoutsuite_venv
    ```

5.  **Activate the Virtual Environment:**
    ```bash
    source ~/scoutsuite_venv/bin/activate
    ```
    *Your terminal prompt will now show `(scoutsuite_venv)` at the beginning.*

### 2.2. Install ScoutSuite into the Virtual Environment

1.  **Upgrade pip within the virtual environment:**
    ```bash
    pip install --upgrade pip
    ```

2.  **Install ScoutSuite:**
    ```bash
    pip install scoutsuite
    ```

3.  **Verify ScoutSuite Installation:**
    ```bash
    scout --help
    ```
    *You should see the ScoutSuite help message.*

### 2.3. Ensure AWS CLI v2 is Available System-Wide

1.  **Deactivate your virtual environment (temporarily):**
    ```bash
    deactivate
    ```
    *Your prompt will return to normal.*
2.  **Verify the system-wide AWS CLI version:**
    *This confirms that AWS CLI v2 is correctly installed and accessible at a system level, which we will use for SSO configuration.*
    ```bash
    aws --version
    ```
    * **Expected Output:** You *must* see aws-cli/2.x.x ... (e.g., aws-cli/2.27.53 Python/3.13.4 ...).
---

## 3. Configuring AWS CLI with IAM Identity Center (SSO)

This is how ScoutSuite will authenticate with your demo AWS account.

### 3.1. AWS IAM Identity Center Pre-requisites

* Ensure **AWS IAM Identity Center** is enabled in your AWS Organization.
* Ensure your **user** is configured in IAM Identity Center.
* Ensure a **Permission Set** (e.g., ScoutSuite-SecurityAudit or ReadOnlyAccess) is assigned to your user for your demo AWS account.

### 3.2. Run the `aws configure sso` Wizard

1.  **Reactivate your virtual environment:**
    ```bash
    source ~/scoutsuite_venv/bin/activate
    ```
    

2.  **Start the SSO Configuration Wizard:**
    *Since the direct aws configure sso command initially resulted in an "Invalid choice" error (as seen in image_c87c05.png` and `image_c8789f.png), we use the full path to the globally installed AWS CLI v2 to ensure it's invoked correctly.*
    ```bash
    /usr/local/bin/aws configure sso
    ```
    
3.  **Follow the Interactive Prompts carefully:**

    * **SSO session name (Recommended):** Enter a name (e.g., joelscloud-sso).

    * **SSO start URL [None]:** Paste your AWS access portal URL.
        *(Found in IAM Identity Center > Settings > AWS access portal URL)*

    * **SSO region [None]:** Enter the region where your IAM Identity Center instance is located (e.g., `us-east-1`).

    * **SSO registration scopes [sso:account:access]:** Press `Enter`.

    * **Browser Authentication:** A browser window will automatically open.
        * Log in with your IAM Identity Center credentials.
        * Approve access.
        * Close the browser tab and return to the terminal.
   
    * **Select AWS Account ID:** The CLI will list accounts (e.g., `804540873227`).
       
    * **Select Permission Set/Role:** The CLI will list roles. Select your read-only role (e.g., `ScoutSuite-SecurityAudit`).
       
    * **Default client Region [None]:** Enter the region where your demo AWS resources are (e.g af-south-1 or us-east-1).
       
    * **CLI default output format [json]:** Press Enter.

    * **CLI profile name [SuggestedName]:** Accept the suggested name (e.g., ScoutSuite-SecurityAudit-804540873227) or enter a memorable one (e.g., demo-scoutsuite). Then press Enter.

---

## 4. Running the ScoutSuite Scan and Viewing the Report

### 4.1. Sign In to Your SSO Session

1.  **Obtain temporary credentials for your profile:**
    *You must run this command once per session (or when credentials expire) to use the profile.*
    ```bash
    /usr/local/bin/aws sso login --profile <your-chosen-profile-name>
    ```
    *(e.g /usr/local/bin/aws sso login --profile ScoutSuite-SecurityAudit-804540873227)*
   

2.  **Verify credentials:**
    *This confirms that your SSO login was successful and that you have valid temporary credentials.*
    ```bash
    /usr/local/bin/aws sts get-caller-identity --profile <your-chosen-profile-name>
    ```
    * **Expected Output:** JSON showing your account ID and assumed role ARN.*
  

### 4.2. Run the ScoutSuite Assessment

1.  **Execute the ScoutSuite scan:**
    *Now that your AWS CLI is authenticated via SSO, ScoutSuite can use those credentials.*
    ```bash
    scout aws --profile <your-chosen-profile-name>
    ```
    *(e.g. scout aws --profile ScoutSuite-SecurityAudit-804540873227)*
    *The scan will run, showing progress in the terminal.*

### 4.3. View the ScoutSuite Report

1.  **Locate the Report Directory:**
    * Once the scan completes, ScoutSuite will output the path to the report. It's typically scoutsuite-report-aws-YYYYMMDDHHMMSS in your home directory (/home/tyrone/).
    * Navigate into that directory:
        ```bash
        cd ~/scoutsuite-report-aws-YYYYMMDDHHMMSS
        ```

2.  **Open the Report in the Web Browser:**
    ```bash
    xdg-open index.html
    ```
    *This will launch your Ubuntu VM's default web browser (e.g Firefox) and display the interactive ScoutSuite report.*

---
