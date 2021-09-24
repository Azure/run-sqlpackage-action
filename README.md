# SqlPackage command-line Action

`SqlPackage.exe` is a command-line utility that automates validation and deployment of database projects for SQL Server, Azure SQL Database and Azure Synapse. This actions provides support for implement CI/CD workflows on GitHub Actions for database projects running on Azure. When combined with `Visual Studio Database Projects`, it provides a convenient way to define the desired state of a database and apply, deploy and evolve the schema in different databases on different environments.

## When to use it

Use `run-sqlpackage` GitHub action to implement CI/CD workflows that automate the evolution of the schema in a database running on Azure. It provides tools to validate changes, detect data integration errors, data loss issues and perform changes in target databases accross different environments. 

## About database projects

You can use database projects to create new databases and to update existing databases enabling you to apply version control and project management techniques to your database development efforts in much the same way that you apply those techniques to managed or native code. You can help your development team manage changes to databases and database servers by creating a database project, or a server project and putting it under version control. Members of your team can then check out files to make, build, and test changes in an isolated development environment, or sandbox, before sharing them with the team. To help ensure code quality, your team can finish and test all changes for a particular release of the database in a staging environment before you deploy the changes into production.

## How SqlPackage and Database Projects work together

The output of a database project is a file in `dacpac` extension, which describes the **final desired state** of a database. `SqlPackage` then can use such file to detect the evolution of the schema and compute the requered changes that need to be performed in a given target database in order to make it match the given **final state described in the dacpac**. By doing that, you keep on asset that describes you database, while `SqlPackage` compute the deltas for each target database that you want to deploy to.

## Getting Started

### Prerequisites

 - An Azure Synapse SQL Pool, Azure SQL Database or Azure SQL Managed Instance.
 - If you are using SQL Authentication
    - A SQL Login created in the target database.
    - The SQL Login and SQL Password.
 - If you are using Azure Active Directory Authentication
    - A Service Principal created in the tenant.
    - A database user created on the target database associated with the mentioned Service Principal. Follow the documentation at [Create contained users mapped to Azure AD identities](https://docs.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure?tabs=azure-powershell#create-contained-users-mapped-to-azure-ad-identities)
    - An authentication token issued for the mentioned Service Principal and targeting the given database. You can use the GitHub Action [azure-resource-login](https://github.com/marketplace/actions/azure-resource-login) to generate such a token.
 - A [Visual Studio Database Project](https://docs.microsoft.com/en-us/sql/azure-data-studio/extensions/sql-database-project-extension?view=sql-server-ver15) created either using Azure Data Studio, VS Code or Microsoft Visual Studio. See [Working with an already existing database](#Working-with-an-already-existing-database) if you want to create a project based on an existing database.
 - A publishing profile in the format of an XML file indicating how deployment should be done. [A sample file](publish.xml) is provided in the repository for your convenience.

#### Working with an already existing database

More often than not you are not starting a new database from scratch, but you already have a database from were you want to start from. If that's the case, then you can build a database project based on such existing database. Follow the documentation [Create a database project from an existing database](https://docs.microsoft.com/en-us/sql/azure-data-studio/extensions/sql-database-project-extension-getting-started?view=sql-server-ver15#create-a-database-project-from-an-existing-database).

### Inputs

| Name | Description | Required |
| --- | --- | --- |
| `action` | Action parameter to run with `SqlPackage`. Supported values are: `Publish`, `DeployReport`, `DriftReport`, `Script`. | true |
| `sourcepath` | The path where to look for the DACPAC file. If multiple files exists, all of them are processed. | true |
| `profile` | The publishing profile path to use during the execution (in xml format). | true |
| <nobr>`database-server`</nobr> | Database server URL (without protocol). If not indicated in the publishing profile, it has to be indicated here. | false |
| <nobr>`database-name`</nobr> | Database name. If not indicated in the publishing profile, it has to be indicated here. | false |
| `authtoken` | The authentication token used to connect to the database, if credentials not indicated in the connection string. | false |
| `outputpath` | The output folder where assets will be generated if any. If not indicated, the current path is used. | false |
| `outputfile` | The output file name. The final name of the file will be [dacpac_name].[outputfile]. The extension of the file depends on the action performed | false |


### Example usage

#### Generate a deployment report

```
id: deploy-report
name: Identifying proposed changes
uses: Azure/run-sqlpackage-action@v1.0.0
with:
   action: 'DeployReport'
   sourcepath: build
   outputpath: reports
   outputfile: 'deployreport.xml'
   profile: profile.xml
   authtoken: ${{ steps.sql-login.outputs.token }}
```

> Note 1: The parameter `authtoken` is provided as the output of another step.

> Note 2: The target database is not indicated in the action but in the profile. Parameters `TargetConnectionString` and `TargetDatabaseName` need to be supplied in the file `profile.xml`.

#### Deploy changes in a target database

```
id: deploy-target
name: Deploying changes to target
uses: Azure/run-sqlpackage-action@v1.0.0
with:
   action: 'Publish'
   sourcepath: build
   profile: profile.xml
   authtoken: ${{ steps.sql-login.outputs.token }}
```
> Note 1: The parameter `authtoken` is provided as the output of another step.

> Note 2: The target database is not indicated in the action but in the profile. Parameters `TargetConnectionString` and `TargetDatabaseName` need to be supplied in the file `profile.xml`.

#### Deploy changes in a target database specified as parameters

```
id: deploy-target
name: Deploying changes to target
uses: Azure/run-sqlpackage-action@v1.0.0
with:
   action: 'Publish'
   sourcepath: build
   profile: profile.xml
   database-server: mysynapse.sql.azuresynapse.net
   database-name: adventureworks
   authtoken: ${{ steps.sql-login.outputs.token }}
```
> Note 1: The parameter `authtoken` is provided as the output of another step.

> Note 2: In this example `adventureworks` is a database inside of the `mysynapse` Synapse Workspace.

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
