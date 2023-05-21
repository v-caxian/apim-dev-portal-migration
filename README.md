# apim-dev-portal-migration

1. Create [Service Connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#create-a-service-connection) in Azure DevOps for each of the APIM instances you want to migrate from and to.

2. Create a [variable group](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=classic#create-a-variable-group) in Azure DevOps called ***apim-dev-portal*** with the following variables:

    | Variable Name | Description |
    | ------------- | ----------- |
    | APIM_INSTANCE_NAME | The name of the APIM instance to migrate from |
    |REPOSITORY_NAME|The name of the Azure DevOps repository to save the artifacts to|
    |RESOURCE_GROUP_NAME|The name of the resource group the APIM instance is in|
    |SERVICE_CONNECTION_NAME|The name of the Azure DevOps service connection to use to connect to the APIM instance|
    |TARGET_BRANCH|The name of the branch to raise the PR against|

3. Create a variable group in Azure DevOps in the format ***apim-dev-portal-{env}*** for each of the environment you want to migrate to with the following variables:

    | Variable Name | Description |
    | ------------- | ----------- |
    | APIM_INSTANCE_NAME | The name of the APIM instance to migrate to |
    |RESOURCE_GROUP_NAME|The name of the resource group the APIM instance is in|
    |SERVICE_CONNECTION_NAME|The name of the Azure DevOps service connection to use to connect to the APIM instance|

4. Create an [environment](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops#create-an-environment) in Azure DevOps for each of the environments you want to migrate to. Add required [gates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops&tabs=check-pass#approvals) to the environment.

5. Create a pipeline in Azure DevOps with the yaml from file ***capture.yaml***. This will create a pipeline that will capture the APIM developer portal snapshot from the source environment and raise a PR against the repository specified in the variable group.

6. Run the pipeline with the following parameters:

    OUTPUT_FOLDER_PATH: The name of the folder to save the snapshot to in the repository.
    
    _Note_: Pipeline will need approval to access the variable group for the first time.

7. Create a pipeline in Azure DevOps with the yaml from file ***release.yaml***. This will create a pipeline that will deploy the APIM developer portal snapshot to the target environment(s).

8. Link the variable groups created for each environment to the pipeline. Edit the pipeline, select Triggers from the ellipsis menu and navigate to the Variables tab. Add the variable groups to the pipeline.

9. Create url overwrite files for each environment in the format ***urls.{env}.json***. The contents of the file should be as
    ```json
    {
        uri: "https://uri1.com, https://uri2.com"
    }
    ```
    Update the existing uris in the snapshot in the file ***existingUrls.json***

    _Note:_ This is an optional step and is only required if you want to overwrite the urls in the snapshot with the urls in the file.     

10. Run the pipeline with the following parameters:

    OUTPUT_FOLDER_PATH: The name of the folder the snapshot was saved to in the repository.

    ENVIRONMENTS: The name of the environments to deploy to. For ex:
    ```
        - dev
        - stage
        - prod
    ```
    These should match the suffix of the names of the variable groups created in step 3.
