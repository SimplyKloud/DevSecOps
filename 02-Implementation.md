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

