OWASP ZAP and its benefits
- Open Web Application Security Project is a non profit organization which released the product ZAP (Zed Attach Proxy)
- OWASP ZAP is an open source web application security scanner
- It can scan both web applications and API specifications

trigger:
- master

pool:
  vmImage: ubuntu-latest

steps:
- script: |
      wget https://github.com/zaproxy/zaproxy/releases/download/v2.16.0/ZAP_2.16.0_Linux.tar.gz
      tar -xvf ZAP_2.16.0_Linux.tar.gz
      cd ZAP_2.16.0
      ./zap.sh -cmd -quickurl https://www.example.com -quickprogress -quickout /home/vsts/work/1/a/zap_report.html 
  displayName: "Integrate DAST scan using OWASP ZAP in ADO DevSecOps Pipeline"

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: zap_report.html


  
