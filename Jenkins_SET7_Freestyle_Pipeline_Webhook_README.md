# JENKINS – COMPLETE PRACTICAL PROCEDURE

**Order:** Freestyle Project → Pipeline Project → GitHub Webhook using ngrok → SET-7 scenario-based questions.

This README follows the Jenkins Freestyle task in SET-7 and the Pipeline/Webhook procedures in the Week 10 Jenkins notes. It uses the existing Jenkins, Java, Maven, Git and ngrok setup; it does not ask you to install anything new.

**SET-7 GitHub repository:**

```text
https://github.com/deepthisagar7/metro-reservation.git
```

---

# PART 1 – JENKINS FREESTYLE PROJECT

**Aim:** Build the Metro Reservation Java web application from GitHub using Maven and generate a WAR file.

## Step 1: Open Jenkins

**WHERE: Windows + Chrome**

1. Open Jenkins using the way you normally start it.
2. Open Chrome.
3. Enter:

```text
http://localhost:8081
```

4. Log in to Jenkins.

## Step 2: Check Java, Maven and Git

**WHERE: Windows CMD**

1. Press `Windows + R`.
2. Type `cmd`.
3. Press Enter.

Run these commands one by one:

**Check Java:**

```bat
java -version
```

**Check Maven:**

```bat
mvn -version
```

**Check Git:**

```bat
git --version
```

If all three display their versions, continue.

## Step 3: Create a Freestyle Project

**WHERE: Jenkins browser**

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter the project name:

```text
MetroReservation_Build
```

4. Select **Freestyle project**.
5. Click **OK**.

## Step 4: Configure the GitHub Repository

**WHERE: Jenkins → MetroReservation_Build → Configure**

1. Find **Source Code Management**.
2. Select **Git**.
3. In Repository URL, paste:

```text
https://github.com/deepthisagar7/metro-reservation.git
```

4. In Branch Specifier, enter:

```text
*/main
```

If the repository uses `master`, use `*/master` instead.

5. If the repository is public, credentials can remain as None.
6. Scroll down.

## Step 5: Configure JDK and Maven

**WHERE: Jenkins Dashboard → Manage Jenkins → Tools**

1. Go to **Manage Jenkins**.
2. Open **Tools** (sometimes shown as Global Tool Configuration).
3. Find **JDK installations**.
4. If a JDK installation is already configured, note its name.
5. Find **Maven installations**.
6. If Maven is already configured, note its name.
7. Click Save if you made changes.

**Important:** Do not install another JDK or Maven. Use the installations already available on your computer. If the tool configuration is already done, leave it as it is.

## Step 6: Add the Maven Build Command

**WHERE: Freestyle project → Configure → Build Steps**

1. Return to your `MetroReservation_Build` configuration.
2. Find **Build Steps**.
3. Click **Add build step**.
4. Select **Execute Windows batch command**.
5. Paste these commands:

```bat
mvn clean
mvn package
```

6. Click **Apply**.
7. Click **Save**.

What these commands do:

- `mvn clean` – removes previous build output.
- `mvn package` – builds the application and creates the WAR file if the project is configured for WAR packaging.

## Step 7: Run the Freestyle Project

**WHERE: Jenkins browser**

1. Open `MetroReservation_Build`.
2. Click **Build Now**.
3. Look at Build History.
4. Click the latest build number.
5. Click **Console Output**.
6. Wait for the build to finish.

Expected output:

```text
BUILD SUCCESS
```

If you see `BUILD SUCCESS`, the build has completed successfully.

## Step 8: Verify the WAR File

**WHERE: Jenkins workspace / File Explorer**

1. Open your Jenkins job.
2. Click **Workspace**, if available.
3. Open the `target` folder.
4. Check whether the WAR file was generated.

Expected filename:

```text
metro-reservation.war
```

If the generated filename differs slightly, check the actual `.war` file inside `target`.

---

# PART 2 – JENKINS PIPELINE PROJECT

**Aim:** Create a Jenkins Pipeline using a script that performs Git clone, clean, install, test and package stages.

## Step 1: Open Jenkins Dashboard

**WHERE: Jenkins browser**

1. Open Jenkins Dashboard.
2. Click **New Item**.
3. Enter the project name:

```text
MetroReservation_Pipeline
```

4. Select **Pipeline**.
5. Click **OK**.

## Step 2: Configure the Pipeline

**WHERE: Pipeline project → Configure**

1. Enter a description if required:

```text
Metro Reservation application pipeline
```

2. Scroll down to the **Pipeline** section.
3. Find **Definition**.
4. Select:

```text
Pipeline script
```

5. A script box will appear.

## Step 3: Enter the Pipeline Script

**WHERE: Pipeline script box**

The following follows the structure in the Week 10 notes. Replace the placeholder GitHub URL with the SET-7 repository URL.

```groovy
pipeline {
    agent any

    tools {
        maven 'MAVEN-HOME'
    }

    stages {

        stage('git repo & clean') {
            steps {
                bat "git clone https://github.com/deepthisagar7/metro-reservation.git"
                bat "mvn clean -f mavenjava"
            }
        }

        stage('install') {
            steps {
                bat "mvn install -f mavenjava"
            }
        }

        stage('test') {
            steps {
                bat "mvn test -f mavenjava"
            }
        }

        stage('package') {
            steps {
                bat "mvn package -f mavenjava"
            }
        }
    }
}
```

**Important before running:**

- `MAVEN-HOME` must match the Maven installation name configured in Jenkins Tools.
- `mavenjava` is the project folder name used in the Week 10 notes. The commands using `-f` must point to the actual folder containing the project's `pom.xml`.
- The script uses Windows `bat` commands, as in the notes.
- Pipeline support must already be available in your Jenkins setup.

## Step 4: Save the Pipeline

**WHERE: Jenkins browser**

1. Scroll to the bottom.
2. Click **Apply**.
3. Click **Save**.

## Step 5: Run the Pipeline

**WHERE: Jenkins browser**

1. Open `MetroReservation_Pipeline`.
2. Click **Build Now**.
3. Open the latest build.
4. Click **Console Output** or **Pipeline Overview**.

You should see these stages executing:

1. **git repo & clean** – clone the GitHub repository and clean previous build output.
2. **install** – install the required Maven dependencies and project artifacts.
3. **test** – run the project's test cases.
4. **package** – package the application and generate the WAR file, if configured for WAR packaging.

Expected final output:

```text
Finished: SUCCESS
```

---

# PART 3 – GITHUB WEBHOOK USING NGROK

**Aim:** Whenever a developer pushes code to GitHub, Jenkins should automatically start a build.

Flow:

```text
Developer pushes code to GitHub
              ↓
        GitHub Webhook
              ↓
     ngrok forwards request
              ↓
  Jenkins automatically starts build
```

## Step 1: Start Jenkins

**WHERE: Chrome**

Open:

```text
http://localhost:8081
```

Make sure Jenkins is running.

## Step 2: Open ngrok Directly

**WHERE: Windows File Explorer**

Use your existing `ngrok.exe`.

1. Press `Windows + E`.
2. Open the folder where you have downloaded/extracted ngrok.
3. Find `ngrok.exe`.
4. Open the folder containing it.
5. Click the address bar at the top of File Explorer.
6. Type:

```text
cmd
```

7. Press Enter.

A Command Prompt will open directly in the ngrok folder.

## Step 3: Start the ngrok Tunnel

**WHERE: The CMD window opened in the ngrok folder**

Run:

```bat
ngrok.exe http 8081
```

Wait until you see:

```text
Session Status: online
```

You should also see a forwarding address similar to:

```text
https://your-generated-address.ngrok-free.dev
```

Copy the actual HTTPS forwarding address displayed on your screen.

**Keep this CMD window open.** Closing it stops the tunnel.

## Step 4: Open GitHub Webhook Settings

**WHERE: Chrome → GitHub**

1. Open:

```text
https://github.com/deepthisagar7/metro-reservation
```

2. Click **Settings**.
3. Click **Webhooks**.
4. Click **Add webhook**.

## Step 5: Configure the Payload URL

**WHERE: GitHub → Add webhook**

In **Payload URL**, paste the HTTPS URL you copied from ngrok. At the end, add:

```text
/github-webhook/
```

For example, if ngrok displays:

```text
https://example.ngrok-free.dev
```

Your complete Payload URL will be:

```text
https://example.ngrok-free.dev/github-webhook/
```

Other settings:

| Field | Value |
|---|---|
| Content type | `application/json` |
| Which events? | Just the push event |
| Active | Checked |

Click **Add webhook**.

## Step 6: Enable the Webhook Trigger in Jenkins

**WHERE: Jenkins browser**

1. Open your Jenkins project.
2. Click **Configure**.
3. Find **Build Triggers**.
4. Tick:

```text
GitHub hook trigger for GITScm polling
```

5. Click **Save**.

Do this for the job you want GitHub to trigger. You can enable it for your Freestyle job or Pipeline job.

## Step 7: Test the Webhook

**WHERE: CMD or PowerShell, inside your local Git repository folder**

Make a small change to a file in your local project and save it.

Then run these commands one by one:

```bat
git add .
```

```bat
git commit -m "Test Jenkins webhook"
```

```bat
git push
```

## Step 8: Verify Automatic Build

**WHERE: Jenkins browser**

1. Open your Jenkins project.
2. Check **Build History**.
3. A new build should start automatically after the GitHub push.
4. Open that build.
5. Check **Console Output**.

**Remember:** Keep ngrok running while testing. If its URL changes after restarting, update the GitHub Payload URL.

---

# PART 4 – SET-7 SCENARIO-BASED QUESTIONS

These are the five questions under **b) Jenkins – Freestyle Project** on page 2 of SET-7.

## 1. What happens if a developer introduces compilation errors into the Metro Reservation application?

**Answer:** Maven compilation fails during the Jenkins build. Jenkins marks the build as FAILED, and the WAR file will not be generated successfully. The developer must correct the errors and push the updated code to GitHub.

## 2. How can Jenkins detect changes made to the GitHub repository?

**Answer:** Jenkins can detect changes using Poll SCM or GitHub Webhooks. Poll SCM checks the repository periodically, whereas a GitHub webhook immediately notifies Jenkins when code is pushed.

## 3. What is the difference between successfully generating the WAR file and deploying the application on Tomcat?

**Answer:**

- **Generating WAR:** Maven packages the application into a `.war` file. It means the application is built.
- **Deploying on Tomcat:** The WAR file is placed in Tomcat and deployed so that users can access the running application through a browser.

## 4. Why is Maven useful for managing dependencies and building the Metro Reservation application?

**Answer:** Maven reads project dependencies from `pom.xml` and downloads the required libraries. It manages the build lifecycle, compiles source code, runs tests and packages the application into a WAR file. It makes the build process easier and consistent.

## 5. How would you troubleshoot a Jenkins build failure caused by an incorrect pom.xml file?

**Answer:**

1. Open Jenkins Dashboard.
2. Open the failed project.
3. Select the failed build number.
4. Click **Console Output**.
5. Identify the error related to `pom.xml`.
6. Check XML tags, dependencies, versions and Maven configuration.
7. Correct the `pom.xml` file.
8. Test the project using:

```bat
mvn clean package
```

9. Commit and push the corrected code to GitHub.
10. Run the Jenkins build again and verify `BUILD SUCCESS`.
