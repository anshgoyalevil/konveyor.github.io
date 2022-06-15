---
title: "Tackle 2.0 Views"
date: 2022-06-14T14:59:53-06:00
draft: true
---

The latest version of Tackle has the Developers view and the new Architects view to support the three main roles of users:
* **Administrators:** Has access to some application-wide configuration parameters that other users can consume but not change or browse.

    Example actions: Define Git credentials, Maven settings, .xml files.

* **Architects:** Often the  technical leads for the migration project that can create and modify applications and information related to it. The Architects do not need to have access to sensitive information, but can consume it.

    Example actions: Associate an existing credential to access the repository of a given application.

* **Migrators:** Developers that can run assessments and analysis, but cannot create or modify applications in the portfolio. Maybe an example action?

## Developer view
The developer view is intended to be used by migrators and stakeholders and has three pages with different functionalities.
* Application inventory
* Reports
* Controls

### Application inventory
The Application inventory page manages the applications being migrated and is where the assessment and analysis processes are performed.
![](/Tackle2/Views/AppInvMainPopulated.png)
The application list provides a holistic view of the application portfolio using an extensible tagging model to classify application types.  The applications can be expanded to show more detailed information which can be edited and managed.

This page has two tabs with different information and functionality: Assessment and Analysis.

#### Application Assessments
Use the Assessment tab to facilitate a conversation before migrating an application with stakeholders such as technical subject matter experts and application owners or users.  Tackle does not prescribe solutions but instead provides a script of discussion points to identify potential migration risks.

Assessment categories include:
* Application details
* Application dependencies
* Application architecture
* Application observability
* Application cross-cutting concerns

Reviewing the results will help stakeholders develop suitable migration strategies for applications or application types. Only one assessment can be done at once, but assessment results can be applied to other applications.

Additional functionality includes:
* Copy assessment
* Copy assessment and review
* Discard assessment
* Manage dependencies
* Manage credentials
* Delete

#### Analysis
Use the Analysis tab to perform an automated examination of the application that views binaries, source code, and dependencies to find possible issues that might prevent the application from running on the new platform. The process starts by retrieving and then analyzing one of the following repositories:
* **Binary:** Provides a full picture of the application and library dependencies that are embedded into the code.
* **Source code:** Only provides a view of the application.
* **Source code + dependencies:** Provides a full picture by downloading the source code from the repository, then parses and downloads the dependencies from Maven and adds them to the source code.

There is an option to upload a locally stored binary. This only works if a single application is selected.

Additional functionality includes:
* Manage dependencies
* Manage credentials
* Analysis details
* Cancel analysis
* Delete

### Reports
The Reports page provides an overview of the assessments and reviews for the entire application inventory.
![](/Tackle2/Views/ReportsMain.png)
The Reports page contains the following sections:

* **Current landscape: **Displays all applications according to their risk levels.
* **Adoption candidate distribution:** Lists the assessed applications with the following columns:
* Criticality is based on the Business criticality value of the review.
* Priority is based on the Work priority value of the review.
* Effort is based on the Effort estimate value of the review.
* Decision is based on the Proposed action value of the review.

 By default, all applications are selected. You can clear some of the application check boxes to filter the report.

* **Suggested adoption plan: **Displays a suggested adoption plan based on effort, priority, and dependencies.
* **Identified risks: **Lists the severe risks identified in the assessments for all applications.

### Controls
The Controls page is where the application parameters are managed by the architect or developers as the instance is configured and edited as the project progresses.  
![](/Tackle2/Views/ControlsMain.png)
The parameters include:
* Stakeholders
* Stakeholder groups
* Job functions
* Business services
* Tags (Tag Types)

## Administrator view
The administrator view is intended to be used by administrators to set up the Tackle instance environment.
![](/Tackle2/Views/AdminView.png)

### Credentials
This management module enables administrators to create and manage credentials for access to private repositories.  It also allows for the architects to assign the credentials to applications without knowing their contents.
![](/Tackle2/Views/CredentialsMainEmpty.png)

The credentials page displays the available credentials with an Edit and Delete buttons and the following fields:
* Name
* Description
* Type
    * Source Control
    * Maven Settings File
    * Proxy
* Type specific information
* Created by

**Note:** Type specific credential information such as keys and passphrases will be hidden or shown as [Encrypted].

#### Creating source control credentials
Follow the steps below to create new credentials for a source control repository.

**Procedure**
1. Click **Credentials** in the left menu of the **Administrator** view.
2. Click the **Create new** button.
![](/Tackle2/Views/NewCredentialSourceControl.png)
3. Enter the following information.
    * Name
    * Description (Optional)
4. Select the **Source Control** in the **Type** drop-down list.
5. Select the **credential type** in the **User Credentials** drop-down list and enter the requested information.
    * Username/Password
        * Username
        * Password (Hidden)
        * Create credentials
    * SCM Private Key/Passphrase
        * SCM Private Key
        * Private Key Passphrase (Hidden)
6. Click **Save**.

Tackle validates the input and creates a new credential. SCM keys must be parsed and checked for validity. If the validation fails, an error message displaying “not a valid key/XML file” is displayed.

#### Creating Maven settings file credentials
Follow the steps below to create new credentials for a Maven repository.

**Procedure**
1. Click **Credentials** in the left menu of the **Administrator** view.
2. Click the **Create new** button.
![](/Tackle2/Views/NewCredentialsMaven.png)
3. Enter the following information.
    * Name
    * Description (Optional)
4. Select **Maven Settings File** in the **Type** drop-down list.
5. Upload the settings file.
6. Click **Save**.

Tackle validates the input and creates a new credential. Maven settings.xml files must be parsed and checked for validity. If the validation fails, an error message displaying “not a valid key/XML file” is displayed.

#### Creating proxy credentials
Follow the steps below to create new credentials for a proxy repository.

**Procedure**
1. Click **Credentials** in the left menu of the **Administrator** view.
2. Click the **Create new** button.
![](/Tackle2/Views/NewCredentialsGeneric.png)
3. Enter the following information.
    * Name
    * Description (Optional)
4. Select **Proxy** in the **Type** drop-down list.
5. Enter the following information.
    * Username
    * Password (Hidden)
6. Click **Save**.

### Repositories
This management module configures the repositories used by Tackle with the following options.
* Git
    * Consume insecure Git repositories
* Subversion
    * Consume insecure Subversion repositories
* Maven
    * Repository size
    * Force update of dependencies
    * Consume insecure artifact repositories

### Proxy
This management module configures HTTP & HTTPS proxy settings and credentials.  To configure the proxies click the radio button and enter the following information.
* Proxy host
* Proxy port
* Authentication (optional)
    * Proxy credentials