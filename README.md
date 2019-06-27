## sps-demo

This is demo for SPS Beijing 2019, how to configure CI/CD in Azure pipeline for SPFx webpart


### Build Pipeline .yaml file:

trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '10.x'
  displayName: 'Install Node.js'

- script: |
    npm install
    npm run build
    npm test
    gulp bundle --ship
    gulp package-solution --ship
  displayName: 'npm install and build and test and pack'

- task: CopyFiles@2
  inputs: 
    contents: '**/*.sppkg'
    targetFolder: $(build.artifactstagingdirectory)
    Overwrite: true

- task: PublishBuildArtifacts@1
  inputs:
    pathtoPublish: $(build.artifactstagingdirectory)
    artifactName: SPS-Demo-SPPKG


### Release Pipeline configuration

- 1, install nodejs 10
- 2. add O365 cli: npm command | Custom | Command and arguments : install -g @pnp/office365-cli
- 3. Connect App Catalog: 
-   o365 spo login https://$(tenant).sharepoint.com/$(catalogsite) --authType password --userName $(username) --password $(password)
- 4. Upload package to App Catalog: (find the path at Advanced -> working directory) 
-   o365 spo app add -p $(System.DefaultWorkingDirectory)/SPSDemoPackage/SPS-Demo-SPPKG/sharepoint/solution/sps-demo.sppkg --overwrite
- 5. deploy solution: 
-   o365 spo app deploy --name spfx-ci-demo.sppkg --appCatalogUrl https://$(tenant).sharepoint.com/$(catalogsite)

- 6. set variables:
-   catalogsite: sites/appcatalog
-   tenant: m365x
-   username: ...@m365x.onmicrosoft.com
-   password: ***

- 7. enable trigger
