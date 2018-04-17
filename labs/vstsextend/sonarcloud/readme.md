---
title: Managing Technical Debt using VSTS and SonarCloud
layout: page
sidebar: vsts2
permalink: /labs/vstsextend/sonarcloud/
folder: /labs/vstsextend/sonarcloud/
---

Last updated : {{ "now" | date: "%b %d,%Y" }}

## Overview

Technical debt is the set of problems in a development effort that make forward progress on customer value inefficient. Technical debt saps productivity by making code hard to understand, fragile, time-consuming to change, difficult to validate, and creates unplanned work that blocks progress. Unless they are managed, technical debt can accumulate and hurt the overall quality of the software and the productivity of the development team in the long term

[SonarCloud](https://about.sonarcloud.io/){:target="_blank"} is an open source platform for continuous inspection of code quality to perform automatic reviews with static analysis of code to:

- Detect Bugs
- Code Smells
- Security Vulnerabilities
- Centralize Quality

### What's covered in this lab

In this lab, you will learn how to integrate Visual Studio Team Services with SonarCloud

- Setup a VSTS project and CI build to integrate with SonarCloud
- Analyze SonarCloud reports
- Integrate static analysis into the VSTS pull request process
- Add a dashboard widget to display the current quality of the project

### Prerequisites for the lab

1. You will need a **Visual Studio Team Services Account**. If you do not have one, you can sign up for free [here](https://www.visualstudio.com/products/visual-studio-team-services-vs){:target="_blank"}

1. A **Microsoft Work or School account, or a GitHub/BitBucket account**. SonarCloud supports logins using any of those identity providers.

## Setting up the Environment

1. Install the SonarCloud VSTS extension to your VSTS account

    - Navigate to the  [SonarCloud extension](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud){:target="_blank"} in the Visual Studio Marketplace and click **Get it free** to install it.

    ![sc_marketplace](images/ex1/sc_marketplace.png)

   {% include important.html content= "If you do not have the appropriate permissions to install an extension from the marketplace, a request will be sent to the account administrator to ask them to approve the installation." %}

   The SonarCloud extension contains build tasks, build templates and a custom dashboard widget.

1. Create a new VSTS project for the lab
    
    - Create a new project in your VSTS account called **SonarExamples**

    - Import the **Sonar Scanning Examples repository** from GitHub at https://github.com/SonarSource/sonar-scanning-examples.git

    ![sc_marketplace](images/ex1/setup_import.png)

    See [here](https://docs.microsoft.com/en-us/vsts/git/import-git-repository?view=vsts){:target="_blank"} for detailed instructions on importing a repository.

    The scanning examples repository contains sample projects for a number of build systems and languages including C# with MSBuild, and Maven and Gradle with Java.

## Exercise 1: Set up a build definition that integrates with SonarCloud

We will set up a new build definition that integrates with SonarQube to analyze the **SonarExamples** code. As part of setting up the build definition we will create a SonarCloud account and organization.

1. In your new VSTS project, go to **Builds** under **Build and Release** tab, then click on **+New** to create a new build definition.

1. Click **Continue** to accept the default values for **source**, **Team project**, **Repository** and **Default branch**

    ![build_source](images/ex1/build_source.png)

   > The SonarCloud extension contains custom build templates for Maven, Gradle, .NET Core and .NET Desktop applications. The templates are based on the standard VSTS templates but with additional analysis-specific tasks and some pre-configured settings.

1. Select the .NET Desktop with SonarCloud template.

    ![build_templates](images/ex1/build_templates.png)

    The template contains all of the necessary tasks and most of the required settings. We will now provide the values for the remaining settings.

1. Select the _Hosted VS2017_ agent queue 

    ![build_config_agentqueue](images/ex1/build_config_agentqueue.png)

1. Configure the _Prepare analysis on SonarCloud_ task

    ![build_config_prepare](images/ex1/build_config_prepare.png)

   There are three settings that need to be configured:

   |Setting|Value|Notes|
   |---------|-----|-----|
   |**SonarCloud Service Endpoint**|SonarCloudSamples|The name of the VSTS endpoint that connects to SonarCloud|
   |**Organization**|_{your SonarCloud org id}_ |The unique key of your organization in SonarQube|
   |**Project Key**|_{your SonarCloud org id}_.netfxdemo |The unique key of the project in SonarQube|

   We will now create the endpoint and an account on SonarCloud.

1. Create a service endpoint for SonarCloud

   - click on the _New_ button to start creating a new endpoint

    ![build_config_prepare_newendpoint](images/ex1/build_config_prepare_newendpoint.png)

1. Create a SonarCloud account

   A service endpoint holds the configuration information VSTS requires to connect to an external service, in this case SonarCloud. There is a custom SonarCloud endpoint that requires two pieces of information: the identity of the organization in SonarCloud, and a token that the VSTS build can use to connect to SonarCloud. We will create both while setting up the endpoint.

   - click on the **your SonarCloud account security page** link

    ![build_config_endpoint](images/ex1/build_config_endpoint.png)

1. Select the identity provider to use to log in to SonarCloud

   As we are not currently logged in to SonarCloud we will be taken to the SonarCloud login page.

   - select the identity provider you want use and complete the log in process

    ![sc_identity_providers](images/ex1/sc_identity_providers.png)

1. Authorize SonarCloud to use the identity provider

   > The first time you access SonarCloud, you will be asked to grant SonarCloud.io access to your account. The only permission that SonarCloud requires is to read your email address

    ![sc_authorize](images/ex1/sc_authorize.png)

    After authorizing and logging in, we will be redirected to the **Generate token** page

1. Generate a token to allow VSTS to access your account on SonarCloud:

   - enter a description name for the token e.g. "vsts_build" and click **Generate** 

   - click **Generate**

    ![sc_generatetoken1](images/ex1/sc_generatetoken.png)

1. Copy the generated token

   - click **Copy** to copy the new token to the clipboard

    ![sc_generatetoken2](images/ex1/sc_generatetoken2.png)


    {% include note.html content= "You should treat Personal Access Tokens like passwords. It is recommended that you save them somewhere safe so that you can re-use them for future requests." %}

   We have now created an organization on SonarCloud, and have the token needed configure the VSTS endpoint.

1. Finish creating the endpoint in VSTS
   - return to VSTS **Add new SonarCloud Connection** page, set the **Connection name** to **SonarCloud** enter the **SonarCloud Token** you have just created.
   - click **Verify connection** to check the endpoint is working, then click **OK** to save the endpoint.

    ![build_config_endpoint_completed](images/ex1/build_config_endpoint_completed.png)

1. Finish configuring the **Prepare analysis on SonarCloud** task.

   - click on the **Organization** drop-down and select your organization.
   - enter a unique key for your project e.g. **[your account].visualstudio.com.sonarexamples.netfx**
   - enter a friendly name for the project e.g. **Sonar Examples - NetFx**

    ![build_config_prepare_completed](images/ex1/build_config_prepare_completed.png)

1. [Optional] Enable the _Publish Quality Gate Result_ step

   This step is not required and is disabled by default.
   If this step is enabled, a summary of the analysis results will appear on the _Build Summary_ page. However, this will delay the completion of the build until the 
   processing on SonarCloud has finished.

1. Save and queue the build.

   ![build_in_progress](images/ex1/build_run_in_progress.png)

1. If you enabled the _Publish Quality Gate Result_ step above the Build Summary will contain a summary of the analysis report. 

   ![build_completed](images/ex1/build_run_completed.png)

1. Either click on the **Detailed SonarQube Report** link in the build summary to open the project in SonarCloud, or browse to SonarCloud and view the project.

   ![sc_analysis_report](images/ex1/sc_analysis_report.png)

   We have now created a new organization on SonarCloud, and configured a VSTS build to perform analysis and push the results of the build to SonarCloud.

## Exercise 2: Analyze SonarQube Reports

Open the **Sonar Examples** project in the SonarQube Dashboard. The dashboard shows a summary of the quality of the project - the number of issues found, the code coverage, and any code duplication. We have just analysed the project for the first time so there is no historical information. However, as we integrated the analysis into a CI build, new analysis results will be continue to be pushed to SonarQube as the code changes, and trend lines will appear on the dashboard.

Under ***Bugs and Vulnerabilities***, we can see a bug has been caught.

  ![sonar_portal](images/ex2/sonar_portal.png)

  The page has other metrics such as ***Code Smells***, ***Coverage***, ***Duplications*** and ***Size***. The following table briefly explains each of these terms.

   |Terms|Description|
   |-----|-----------|
   |**Bugs**|An issue that represents something wrong in the code. If this has not broken yet, it will, and probably at the worst possible moment. This needs to be fixed|
   |**Vulnerabilities**|A security-related issue which represents a potential backdoor for attackers|
   |**Code Smells**|A maintainability-related issue in the code. Leaving it as-is means that at best maintainers will have a harder time than they should making changes to the code. At worst, they'll be so confused by the state of the code that they'll introduce additional errors as they make changes|
   |**Coverage**|To determine what proportion of your project's code is actually being tested by tests such as unit tests, code coverage is used. To guard effectively against bugs, these tests should exercise or 'cover' a large proportion of your code|
   |**Duplications**|The duplications decoration shows which parts of the source code are duplicated|
   |**Size**|Provides the count of lines of code within the project including the number of statements, Functions, Classes, Files and Directories|

  {% include important.html content= "In this example, along with the bug count, a character **C** is displayed which is known as **Reliability Rating**. **C** indicates that there is **at least 1 major bug** in this code. For more information on Reliability Rating, click [here](https://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Reliability)" %}

1. Click on the **Bugs** count to see the details of the bug.

   ![sonar_portal](images/ex2/sonar_portal.png)

   ![bug_details](images/ex2/bug_details.png)

1. You will see the error in line number 9 of *Program.cs** file as **Change this condition so that it does not always evaluate to 'true'; some subsequent code is never executed.**.

   ![bug_details_2](images/ex2/bug_details_2.png)

   We can also see which lines of code are not covered by tests.

   ![bug_details_code_coverage](images/ex2/bug_details_code_coverage.png)

   Our sample project is very small and has no historical data. However, there are thousnads of public projects on SonarCloud that have more interesting and realistic results.

1. Browse public projects on [SonarCloud](https://sonarcloud.io/projects){:target="_blank"}

   As we are still logged in, we will only see our projects by default.
   - click **Explore** to see all projects

   ![sc_projects_mine](images/ex2/sc_projects_mine.png)

   ![sc_projects_all](images/ex2/sc_projects_all.png)

1. Filter projects by language and size

   - select **C#** in the set of filters
   - select **Size** in the **Sort by:** drop-down
   - sort the results in descending order

   ![sc_projects_all](images/sc_projects_all_filtered.png)

   We have seen how to browse the project on SonarCloud to look at issues that already exist in the code base. Next, we will set up integration with pull requests so that issues can be identified and corrected before they are merged.


## Exercise 3: Set up pull request integration

**TODO**

## Exercise 4: Add a dashboard widget

**TODO**

## Exercise 5: Configure Quality Gate

**TODO - recreate images for SonarCloud**

   Let us create a Quality Gate to enforce a policy which fails the gate if there are bugs in the code. A Quality Gate is a PASS/FAIL check on a code quality that must be enforced before releasing software.

1. Log in to SonarCloud and navigate to the **Sonar Examples** project

1. Click on **Quality Gates** menu and click **Create** in the Quality Gates screen.

   ![qualitygate](images/qualitygate.png)

   ![qg-create](images/qg-create.png)

1. Enter name for the Quality Gate and click **Create**.

   ![qgcreate-popup](images/qgcreate-popup.png)

1. Let us add a condition to check for the number of bugs in the code. Click on **Add Condition** drop down and select the value **Bugs**.

   ![qg-bugs](images/qg-bugs.png)

1. Change the **Operator** value to **is greater than** and the **ERROR** threshold value to **0** (zero) and click on the **Add** button.

   {% include note.html content= "This condition means that if the number of bugs in Sonar Analysis is greater than 0 (zero) then the quality gate will fail and will appear red." %}

   ![qgbug-add](images/qgbug-add.png)

1. To enforce this quality gate for **Sonar Examples** project, click on **All** under **Projects** section and select the project checkbox.

   ![qg-selectproject](images/qg-selectproject.png)

1. Now return to VSTS and queue a new build

   ![TODO](images/build_configure.png)

**TODO**
1. You will see that the build has failed since the associated  **SonarQube Quality Gate** has **failed**. The  count of bugs is also displayed under **SonarQube Analysis Report**.

   ![build_summary](images/build_summary.png)

1. Click on the **Detailed SonarQube Report** link in the build summary to open the project in SonarQube.

   ![analysis_report](images/analysis_report.png)


1. You will see the error in line number 28 of **LoginServlet.java** file as **Make "List" serializable or don't store it in the session**.

   ![bug_details_2](images/bug_details_2.png)

1. The error is because the session attribute accepts only serialized objects. This can be fixed by explicitly casting the list object to serializable. Lets fix this bug -

   Go to below path to edit the file in **VSTS** code tab:-

   >src/main/java/com/microsoft/example/servlet/LoginServlet.java

   Make the following changes in the code as shown:

   - Go to line number **28** and replace the existing code with below snippet.

      >session.setAttribute("employeeList", (Serializable)fareList);

      ![code_edit](images/code_edit.png)

   - Import the below package.

      >import java.io.Serializable;

      ![code_import](images/code_import.png)

1. Commit the changes.

1. Once the CI build completes, you will see the Quality Gate as **Passed** in the build summary along with a brief view of **Test Results**, **Code Coverage** and link to SonarQube Analysis Report.

   ![build_summary_bug_fix](images/build_summary_bug_fix.png)

1. Go to SonarQube portal. You will see the bug count is **0**.

   ![bug_fix_sonar_portal](images/bug_fix_sonar_portal.png)

## Summary

With  the **SonarCloud** extension for **Visual Studio Team Services**, you can embed automated testing in your CI/CD pipleine to automate the measurement of your technical debt including code semantics, testing coverage, vulnerabilities. etc. You can also integrate the analysis into the VSTS pull request process so that issues are discovered before they are merged.
