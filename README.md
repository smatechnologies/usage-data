# Usage Data

This application is a tool designed to make it simple to provide SMA Technologies with daily task counts that a customer runs in their OpCon environments. It runs as an OpCon job (daily, by default) that can be configured like any other job.

By default, it publishes the data to an Azure service and a local JSON file. It is configurable to turn off either publish location.

## Installation

To use the Usage Data tool, simply browse to the [latest release](https://github.com/smatechnologies/usage-data/releases/latest) of this repository and look for the assets. If OpCon is installed on Windows, please download winbase.zip. If OpCon runs in a Docker container, download dockerbase.zip. Also, download the [excludes file](https://github.com/smatechnologies/usage-data/blob/master/config/Executables.json). That is all you need.

**If you are on Windows:**
- Create a folder named **"TaskCollection"** at the same level as **"SAM"** installation folder within OpCon installation. Unzip the winbase.zip archive to that folder.
- E.g. locations - C:\Program Files\OpConxps\TaskCollection, or D:\MyAPplications\OpConxps\TaskCollection
- Create a folder named **"TaskCollection"** in the **"ProgramData"** folder for OpCon at the same level as **"SAM"**. Copy the downloaded **"Executables.json"** file into that folder.
- E.g. location - C:\ProgramData\OpConxps\TaskCollection
**If you are on Docker:**
- Unzip the dockerbase.zip archive to a folder named "TaskCollection" on the host machine.
- From one level above this folder, use **"docker cp TaskCollection <container_name>:/app** to copy the application to the container.
- Alternatively, you may map the above folder to the container when starting your container (if OpCon is not yet running). **E.g.: "docker run ... -v C:\OpCon\TaskCollection:/app/TaskCollection ..."**
- E.g. location - /app/TaskCollection
- Copy the downloaded **"Executables.json"** file into /app/TaskCollection.

## Instructions
- Once downloaded and extracted, open a command prompt to the "TaskCollection" folder on Windows and run "TaskCount.exe", which runs and creates a schedule "SMATaskCollection" and a single job in it "SMATaskCount".
- Alternatively, you may double-click the application, which does the same. However, please note that if there are any errors, they are output to the console, and may not be visible, if you double-click.
- When run without any arguments, the application creates the auto-built schedule and job with all default settings. You may choose to have different settings given below.
- Please note that the job's command line uses a global property called "[[SMAOpConPath]]", which must exist and should have the path to the root OpCon folder, like C:\Program Files\OpConxps.
- When running on Docker, enter the container and go to **/app/TaskCollection** and then run **"TaskCount"** from the prompt to use defaults to set up the schedule and job.

## Settings

You may provide arguments to the application when running it for the first time from the command prompt. These settings will be applied to the job created.

- **SELF_UPDATE:** This argument specifies if the application should update itself if a new version is available on GitHub, before running **(Default "True")**. E.g.: TaskCount.exe SELF_UPDATE=True
- **PUBLISH_TO_CLOUD:** This argument specifies if the task counts should be published to the Azure SQL Database **(Default "True")**. E.g.: TaskCount.exe PUBLISH_TO_CLOUD=False
- **PUBLISH_LOCAL:** This argument specifies if the task counts should be published to a local JSON file **(Default "True")**. E.g.: TaskCount.exe PUBLISH_LOCAL=True
- **PUBLISH_FREQUENCY:** This argument specifies the frequency of the job **(Default "Daily")**. E.g.: TaskCount.exe PUBLISH_FREQUENCY=Weekly
- **MACHINE_NAME:** This argument applies only when setting up the job. This is the OpCon machine name that will be used to run the job (it must be an agent for the local machine where SAM is running). By default the application searches the database for a machine with IP=127.0.0.1, or the local machine's IP or the local machine's fully qualifixed domain name (FQDN).

## How it Works

When the job runs, it first attemps to update itself from the latest version on GitHub, if configured to do so. **Please note that port 443 must be open to talk to https://github.com/smatechnologies/usage-data/releases/latest**.

Then it reads history from the database that SAM is configured to connect to and only fetches all the daily task counts. It does not count null jobs, or job reruns with the same arguments as additional tasks. It also does this without locking tables, so it should not affect SAM's processing.

The application also has the ability to count certain processes (executables) separately, so they can be subtracted from the daily counts, if needed. This list is automatically maintained in the cloud.

After that, it creates a local JSON file with all the daily counts it has along with a breakdown by platform and department. The first time the job runs, it collects all daily counts for the last 1 year. After then, it only collects counts from the last point it left off.

If configured, it also publishes these counts to the cloud database. **Please note that port 443 must be open to talk to https://usagedata.smatechnologies.com**.

## Feedback

Please provide feedback through standard channels.

Enhancement ideas should be submitted via the [SMArt ideas portal](https://smartideas.featureupvote.com/suggestions/add).

To report an issue, please use the [Customer Community](https://smatechnologies.force.com/smartusers/login]) and open a Support case.  We will ask you to be on the latest version.  If you are encountering issues please check here to verify you have the latest!

## Release Notes

### Version 1.2.0
#### Fixes
- Fixed an issue where task collection would skip a day every alternate day of collection.

### Version 1.1.0
#### Fixes
- Fixed an issue where an OpCon user assigned to a role with "inherit all schedules" privilege was unable to see the new task collection schedule.

### Version 1.0.0
#### New Features
- Create a task collection application that when run from the command line without parameters, configures itself by creating an OpCon schedule/job that runs each day and produces a simple job output with the total count of tasks run each day.
- Allows the task count job to publish tasks data to a local JSON file in the application's "ProgramData" directory, which includes counts by day and by department and platform per day.
- The task count application is able to collect data only from the time since the job was last run, so it fetches less data and is hence better performing.
- Allow posting the task counts to the Azure cloud, so it can provide a better view of the tasks run each day/month at customer OpCon environments.
- The task count application is now able to self-update to the latest version from the hosting location.
