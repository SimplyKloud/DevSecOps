Sonarcloud is used for
  -  code quality check
  -  security issues check

Sonarcube needs to be installed on onpremise server and then can be integrated with pipelines for service consumption.
Sonarcloud is SaaS model where we need to just consume the cloud service.

Create a account in solarcloud
1) Java 11 builds will not be accepted by SonarCloud and new builds should start with Java 17
2) sonar.login option will be replaced with sonar.token

Create new organization 
Provide organisation key
then create organisation
analyse new project
provide the project key
Create a token with following steps
  clickon account profile in right top corner
  click on security
  provide a token and generate token

copy these token value, org name, host name, project key and provide in pipeline yaml file.

https://sonarcloud.io/create-organization?selectedAlm=manual >> login using github account

dotnet sonarscanner begin \
  /o:"simplykloud" \
  /k:"simplykloud_projectkey" \
  /d:sonar.host.url="https://sonarcloud.io" \
  /d:sonar.token="d4376e429592705f037d8833e8fefbfe44fe1d20"

# SAST
## Quality gate Implementation
1) Create Custom Quality Gate in SonarCloud and Add conditions to the Quality Gate
2) Assign this Quality Gate to the Project
3) Add script in azure-pipeline.yaml file to enable quality gate check (Note: This will fail your build in case Quality Gate fails)

sleep 5
sudo apt update
sudo apt install curl jq 
quality_status=$(curl -s -u 14ad4797c02810a818f21384add02744d3f9e34d: https://sonarcloud.io/api/qualitygates/project_status?projectKey=azuredevopsadodevsecopsprojectkey | jq -r '.projectStatus.status')
echo "SonarCloud Analysis Status is $quality_status"; 
if [[ $quality_status == "ERROR" ]] ; then exit 1;fi


-----------Sample JSON Response from SonarCloud or SonarQube Quality Gate API---------------------

{
	"projectStatus": {
		"status": "ERROR",
		"conditions": [
			{
				"status": "ERROR",
				"metricKey": "coverage",
				"comparator": "LT",
				"errorThreshold": "90",
				"actualValue": "0.0"
			}
		],
		"periods": [],
		"ignoredConditions": false
	}
}

**azure-pipeline.yaml**

trigger:
- master

pool:
  vmImage: ubuntu-latest

jobs:
- job: run_sonarcloud_with_java_17
  displayName: 'Run SonarCloud Analysis with Java Version 17'
  steps:
  - task: JavaToolInstaller@0
    inputs:
      versionSpec: '17'
      jdkArchitectureOption: 'x64'
      jdkSourceOption: 'PreInstalled'

  - script: |
      sudo apt-get update
      sudo apt-get -y install curl jq
      mvn verify package sonar:sonar -Dsonar.host.url=https://sonarcloud.io/ -Dsonar.organization=azuredevopsdevsecopsoorg -Dsonar.projectKey=azuredevsecopsoprojectkey -Dsonar.token=d268968115b4935bcd007ff54f01906c18e33cb6
      sleep 5
      quality_status=$(curl -s -u d268968115b4935bcd007ff54f01906c18e33cb6: https://sonarcloud.io/api/qualitygates/project_status?projectKey=azuredevsecopsoprojectkey | jq -r '.projectStatus.status')
      echo "SonarCloud analysis status is $quality_status"; 
      if [[ $quality_status == "ERROR" ]] ; then exit 1;fi
    displayName: "Integrate SAST using SonarCloud to populate code coverage in Azure DevOps DevSecOps Pipeline"

## Using variables for token
trigger:
- master

pool:
  vmImage: ubuntu-latest

variables:
- group: "SECURE_TOKENS"
**- name: sonar_token
  value: $[variables.SONARTOKEN]**

jobs:
- job: Sonar_SAST_Scan_Job
  container: maven:3.8.1-openjdk-17-slim
  steps:
  - script: |
      mvn verify package sonar:sonar -Dsonar.host.url=https://sonarcloud.io/ -Dsonar.organization=azuredevopsdevsecopsoorg -Dsonar.projectKey=azuredevsecopsoprojectkey **-Dsonar.token=$(sonar_token)**
    displayName: "Integrate SAST using SonarCloud to populate code coverage and secure pipeline variable in Azure DevOps DevSecOps Pipeline"

Devops >> Pipelines >> Library >> variable group
create a group called "SECURE_TOKENS" // same as pipeline yaml file
add varialble
Varialble name - SONARTOKEN and paste the value
Save
