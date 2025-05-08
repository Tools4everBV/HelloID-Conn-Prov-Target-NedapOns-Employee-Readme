# HelloID-Conn-Prov-Target-NedapOns-Employee

> [!IMPORTANT]
> This repository contains the connector and configuration code only. The implementer is responsible to acquire the connection details such as username, password, certificate, etc. You might even need to sign a contract or agreement with the supplier before implementing this connector. Please contact the client's application manager to coordinate the connector requirements.

> [!NOTE]
 >This connector has limited support for Nedap employees. Please check the [Fact Sheet](#Fact-Sheet) for more details.
<br>Extensive knowledge of HelloID provisioning and Nedap Ons (Nedap user and Nedap employee) are required.


<p align="center">
  <img src="https://user-images.githubusercontent.com/68013812/94918899-c672c700-04b3-11eb-9132-7125bbf77fa5.png"
   alt="drawing" style="width:300px;"/>
</p>

> [!IMPORTANT] Upgrade warning!
> Since the November 14, 2024 update, the connector has been changed from using a single mapping file to three separate files. If a customer prefers not to use three separate mapping files, they can continue to use the previous NedapEmployeeMapping.csv file for all three mappings by adding the same file three times in the configuration.


## Table of contents

- [HelloID-Conn-Prov-Target-NedapOns-Employee](#helloid-conn-prov-target-nedapons-employee)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Getting started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Connection settings](#connection-settings)
    - [Correlation configuration](#correlation-configuration)
    - [Available lifecycle actions](#available-lifecycle-actions)
    - [Field mapping](#field-mapping)
    - [Script Mapping](#script-mapping)
      - [The Primary Contract Calculation foreach employment](#the-primary-contract-calculation-foreach-employment)
      - [Script Mapping lookup values](#script-mapping-lookup-values)
      - [Cluster Configuration](#cluster-configuration)
  - [Remarks](#remarks)
    - [Correlation](#correlation)
    - [Limited Employee Connector](#limited-employee-connector)
    - [Primary Contract](#primary-contract)
    - [Combination Nedap User Connector](#combination-nedap-user-connector)
    - [IO Import validation](#io-import-validation)
    - [Preview Mode](#preview-mode)
    - [OutputInfo](#outputinfo)
    - [Action Details (Script logic explanation)](#action-details-script-logic-explanation)
    - [Compare](#compare)
    - [Always Update](#always-update)
    - [Notifications IsCreated | IsDeleted](#notifications-iscreated--isdeleted)
      - [Example Configuration: ](#example-configuration-)
        - [Custom Events:](#custom-events)
        - [Custom Event Configuration:](#custom-event-configuration)
        - [Notification Configurations:](#notification-configurations)
  - [Mapping Remarks](#mapping-remarks)
    - [Example mappings](#example-mappings)
    - [Three Mapping Types](#three-mapping-types)
    - [Cluster Mapping](#cluster-mapping)
  - [Employee Additional Mapping:](#employee-additional-mapping)
  - [Supported Properties](#supported-properties)
  - [Fact Sheet](#fact-sheet)
  - [Development resources](#development-resources)
    - [API endpoints](#api-endpoints)
    - [API documentation](#api-documentation)
  - [Getting help](#getting-help)
  - [HelloID docs](#helloid-docs)

## Introduction
This Repository does only contains the readme. The source code can be found in a private Repository and is meant only for internal use. Link to Repository: [Nedap Ons Employee](https://github.com/Tools4everBV/HelloID-Conn-Prov-Target-NedapONS-Employee)

Nedap Ons provides an API and XML import method to programmatically interact with its services and data. This connector has limited support for managing employees in Nedap. It does not support (employee nor client) scheduling or other none Identity and Access management functionality. It can create multiple employees accounts in Nedap for each employment (EmployeeId and Employment sequence number combination).

## Getting started

### Prerequisites
- **Nedap PFX Certificate** <br>
  A valid Nedap certificate (.PFX) *Tools4ever need to requests a certificate by Nedap to access the REST API*
- **Credentials IOImport**<br>
  Credentials for the IO Import *Different account credentials as the REST API*
- **Mapping Files**<br>
  Mapping between HR departments/functions to Cluster/ Education/ registrationProfile
- **Custom HelloId Property**<br>
  A custom property on the HelloID Person contract with a combination of the employeeCode and EmploymentCode named: [custom.NedapOnsIdentificationNo]
Example:
  ```javascript
  function getValue() {
      return sourceContract.PersonCode + "-" + sourceContract.EmploymentCode
  }
  getValue();
  ```

### Connection settings

The following settings are required to connect to the API.

| Setting                       | Description                                                                  | Mandatory |
| ----------------------------- | ---------------------------------------------------------------------------- | --------- |
| Environment (Rest)            | The Nedap Environment of the API (Development, Staging, Production )         | Yes       |
| Certificate (.PFX) Path       | <FullPath to Certificate> Nedap-Cert.pfx                                     | Yes       |
| Certificate Password          | Password of the certificate                                                  | Yes       |
| IO ImportUrl                  | The URL to the IOImport, Example: "https://<.ioservice.net/importws/import"  | Yes       |
| IO ImportUsername             | The UserName to connect to the IO Import                                     | Yes       |
| IO ImportPassword             | The Password to connect to the IO Import                                     | Yes       |
| CSV Mapping Weekkaart         | The Path to the mapping file Weekkaart                                       | Yes       |
| CSV Mapping Deskundigheid     | The Path to the mapping file Deskundigheid                                   | Yes       |
| CSV Mapping Teams             | The Path to the mapping file Teams                                           | Yes       |
| CSV Separation Character      | Mapping File CSV Separation Character                                        | Yes       |
| Skip mapping for cluster      | Don't use a mapping file for Teams, but directly use HelloID contract values | Yes       |
| Encoding for the csv imports. | Default value is 'Default'                                                   | Yes       |

### Correlation configuration

The correlation configuration is used to specify which properties will be used to match an existing account within _NedapOns-Employee_ to a person in _HelloID_.

| Setting                   | Value   |
| ------------------------- | ------- |
| Enable correlation        | `True` |
| Person correlation field  | `n.a.`     |
| Account correlation field | `n.a.`     |

> [!IMPORTANT]
> Correlation should be enabled even when no direct person properties are used for correlation. When correlation is not enabled, a correlated account will not be updated during account correlation. The Update action will not be triggered when Correlation is disabled. See remark [Correlation](#correlation).

> [!NOTE]
> In HelloID, you cannot clear the person and account correlation fields. However, the selected properties are not used in the connector.

> [!TIP]
> _For more information on correlation, please refer to our correlation [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems/correlation.html) pages_.

### Available lifecycle actions

The following lifecycle actions are available:

| Action             | Description                                                                     |
| ------------------ | ------------------------------------------------------------------------------- |
| create.ps1         | Creates multiple employee objects for each Employment (employeeId + SequenceNumber), based on the contracts in condition from the Business Rules.                                                                                        |
| delete.ps1         | Removes an existing account or entity.                                          |
| update.ps1         | - Update attributes of the corresponding employee account as configured in the fieldMapping and in the CSV Mappings. <br> - Create new employee account for each Employment (employeeId + SequenceNumber) combination.<br>  - Disable account reference from Account Reference  -*See delete action*-                                                                                               |
| configuration.json | Contains the connection settings and general configuration for the connector.   |
| fieldMapping.json  | Defines mappings between person fields and target system person account fields. |

### Field mapping

The field mapping can be imported using the _fieldMapping.json_ file. Besides the FieldMapping, some properties require additional mappings and are not listed in the field mapping. These properties will be added in the code:
- MutationAction
- Contract
- Deskundigheid
- Cluster
- weekkaart

### Script Mapping
Besides the configuration and field mapping, you can also configure script variables to decide which property from the HelloID contracts is used to look up a value in the mapping tables and how the primary contract calculation should be done. Please note that some `same` configuration must be applied in both the create and update scripts, as shown below:

#### The Primary Contract Calculation foreach employment
The Primary Contract Calculation is, by default, the same as the HelloID Primary Contract calculation. However, the order of properties can be adjusted to meet the customer's needs.

```Powershell
## Configuration
# Primary Contract Calculation foreach employment
$firstProperty =  @{ Expression = { $_.Details.Fte };             Descending = $true  }
$secondProperty = @{ Expression = { $_.Details.HoursPerWeek };    Descending = $true  }
$thirdProperty =  @{ Expression = { $_.Details.Sequence };        Descending = $true  }
$fourthProperty = @{ Expression = { $_.EndDate };                 Descending = $true  }
$fifthProperty =  @{ Expression = { $_.StartDate };               Descending = $false }
$sixthProperty =  @{ Expression = { $_.ExternalId };              Descending = $false }

# Priority Calculation Order (High priority -> Low priority)
$splatSortObject = @{
    Property = @(
        $firstProperty,
        $secondProperty,
        $thirdProperty,
        $fourthProperty,
        $fifthProperty,
        $sixthProperty)
}
```

#### Script Mapping lookup values
```Powershell
# Lookup value which is used in the mapping to determine the education, registration profile, and optionally the Cluster(Team)
$eduRegTeamPrimaryLookupKey   = { $_.Title.ExternalId }         # Mandatory
$eduRegTeamSecondaryLookupKey = { $_.Department.ExternalId }   # Not Mandatory
```

#### Cluster Configuration
```Powershell
# Determine if the cluster requires a CSV mapping or has a one-to-one relationship with the HelloID configuration.
if (-not ($actionContext.Configuration.skipCsvMappingForCluster -eq $true)) {
    $clusterId = { $_.Department.ExternalId }
    $clusterName = { $_.Department.DisplayName }
}
```

## Remarks
### Correlation
Because Nedap contains multiple accounts per HelloID person, the accounts will be correlated based on a custom Contract property named `custom.NedapOnsIdentificationNo`. An example can be found in the [Prerequisites](#prerequisites). This is because HelloID does not support correlation based on Contract properties. Even though we cannot use the Person correlation configuration directly from HelloID, the connector correlation should be enabled. Otherwise, you will lose the update trigger when an account is correlated, and the correlated accounts will not be directly updated after correlation.

### Limited Employee Connector
This connector only supports a limited Employee object. When the required functionality is outside scope of this limited Employee connector, you will need an external supplier to configure a direct connection between your HRM system and Nedap Employees. It this situation, we can still manage your Ons user accounts and permissions.

### Primary Contract
Since the connector supports multiple accounts per a single HelloID person. The default Primary contract calculation is not always applicable. The connector contains an example of a "Primary contract calculation" , that calculates the primary contract for each employment (Nedap Account).

### Combination Nedap User Connector
You can use this connector in combination with the Nedap-Users connector. When you use them both in the same environment. You must add the Employee Target system as "Use account data from system". To make sure the employee object exists in Nedap before starting to create the account object.

### IO Import validation
When updating an employee account, you cannot verify if a property is successfully updated in Nedap. To do this you must check the Import Rapportage from the Nedap UI. *Beheer > Import > Importrapportage inzien*

### Preview Mode
Note that in preview mode (DryRun), all HelloID contracts of a Person are in scope. Therefore, it does not simulate the actual outcome when it comes to determining which account should be created, updated, or deleted. However, this DryRun mode is added to verify if the mapping, configuration settings, etc. are present and correct. The contracts in scope are normally configured in the business rules. This cannot be stimulated in Preview.

### OutputInfo
In the field mapping, some properties have a prefix `_outputInfo`. These properties are **only** used to return data from the Connector, and the information can be seen in the account data or in the notification. Since there can be multiple accounts, the connector returns the values comma-separated. There is no link between the values.
- Cluster
- Education
- NedapOnsIdentificationNo
- RegistrationProfile
- isDeleted
- isCreated

> The connector relies on the properties inside _outputInfo. When a specific outputInfo is not required and is removed from the field mapping, the code might need to be adjusted to prevent issues.

### Action Details (Script logic explanation)
The connector is designed using the following steps: First, it calculates and verifies the actions for each desired account and stores the necessary information to create, update, or delete in an ActionDetails list. Once all the information is gathered, the connector proceeds with the standard process to execute the actions using the ActionList for each account.

### Compare
There is a mismatch between the account object from the REST API and the IOImport. To compare the account properties, the connector uses a hardcoded object to compare the properties against the `ActionContext.Data`. This means that when you add a new property in the FieldMapping, you should also remember to add the property to this hardcoded comparison object.

> Keep in mind that only the properties that can be retrieved from Nedap are compared and displayed in the audit logs.

### Always Update
The Update action always updates the Nedap account because some properties cannot be compared with the existing account.

### Notifications IsCreated | IsDeleted
The connector has two properties: IsCreated and IsDeleted. These properties are used for custom notifications. The connector cannot use the standard notification because there are multiple accounts per person. Therefore, it is possible that an account is created in the Update script, or during creation, one of the two accounts is correlated, triggering an account update. This means there will be no standard creation trigger

The properties are populated with the accounts that are created or deleted, respectively, in the Create, Update, and Delete actions. When there are no creations or deletions, it returns  `null`. Therefore, you will need multiple custom events to cover all cases. You can also use the `Has Value` filter. Example notification message: *Created accounts with the following NedapOnsIdentificationNo's: [{{DATA._outputInfo.isCreated}}]*

#### Example Configuration: <br>
##### Custom Events:
 <img src="assets/Custom Events.png">

##### Custom Event Configuration:
 <img src="assets/Custom Events Configuration.png">

##### Notification Configurations:
 <img src="assets/Notification Configurations.png">



> [!TIP]
> Read more about [Custom Notification](https://docs.helloid.com/en/provisioning/notifications--provisioning-/custom-notification-events--conditional-notifications-.html).

## Mapping Remarks
### Example mappings
Example mappings can be found in the Assets folder.

### Three Mapping Types
The connector is built containing three properties which cannot be mapped directly from the person model in HelloID. So there are a multiple mapping file required. These files must include a mapping between HR departments and/or function to a Nedap Cluster, Education or Registration profile. The actions of the connector fails, when there is no mapping found. Of course, this can be changed in the code. But is not configurable by default.

### Cluster Mapping
The Teams/Cluster mappings are often a one-to-one relationship with the HelloID data model, meaning no mapping file is required. The connector offers a switch in the configuration that determines whether to use HelloID values directly. When using direct values, there is also a specific mapping that determines when these values should be used, as shown [Cluster Configuration](#cluster-configuration)


> [!TIP]
> Please note that, starting with version 2.0.0, the connector was changed from using a single mapping file to three separate files. If a customer does not want to use three separate mapping files, they can use the previous `NedapEmployeeMapping.csv` file for all three mappings by simply adding the same file three times in the configuration.


## Employee Additional Mapping:
| Header                       | Description                                             |
| ---------------------------- | ------------------------------------------------------- |
| Primary Contract Calcuatlion | Example of a Primary contract calculation.              |
| Title.ExternalId             | Property of the HelloID primary contract per employment |
| Department.ExternalId        | Property of the HelloID primary contract per employment |
| EducationId                  | Deskundigheidsprofiel > Import Code                     |
| RegistrationProfile          | Weekkaartprofiel > ProfileName                          |
| ClusterId                    | Organigram > Identificatie                              |
| ClusterName                  | Organigram > Naam                                       |

> [!TIP]
> That the mapped value will be created if they do not exist in Nedap! So if you choose in a RegistrationProfile that does not exist, it will be created. What might encounter some unexpected behavior.*

## Supported Properties
| PropertyName         | Notes                                            |
| -------------------- | ------------------------------------------------ |
| Id                   | IdentificationNo                                 |
| Firstname            |                                                  |
| Birthname            |                                                  |
| Lastname             |                                                  |
| DateOfBirth          |                                                  |
| EmailAddress         |                                                  |
| Gender               |                                                  |
| Initials             |                                                  |
| NameUsage            |                                                  |
| AuthenticationNumber | Is needed for second  factor                     |
| Education            | Deskundigheidsprofiel => Import Code             |
| RegistrationProfile  | Weekkaartprofiel => ProfileName                  |
| Cluster              | Organigram > Identificatie (Team)                |
| Contract             | Only a single Employment. With start and EnDate. |

## Fact Sheet
The following table displays an overview of the functionality for the Nedap Ons connector for HelloID Provisioning and Service Automation.

| Type of action                                           | Nedap | HelloID provisioning                             | HelloID Service Automation |
| -------------------------------------------------------- | ----- | ------------------------------------------------ | -------------------------- |
| Create Employees                                         | Yes   | Yes, *Simple employee, no scheduling*            | No                         |
| Update Employees                                         | Yes   | Yes, *Simple employee, no scheduling*            | No                         |
| Delete Employee                                          | No    | No, *Sets an enddate on contract*                | No                         |
| Manage Employee Contracts                                | Yes   | Yes 1 contract, *Simple employee, no scheduling* | No                         |
| Set RegestrationProfiel (Weekkaart)                      | Yes   | Yes, *Additional mapping required*               | No                         |
| Set Cluster (Team)                                       | Yes   | Yes, *Additional mapping required*               | No                         |
| Set education (Deskundigheidsprofiel )                   | Yes   | Yes, *Additional mapping required*               | No                         |
| Set Dashboard Profile                                    | No    | No                                               | No                         |
| Set Discipline                                           | Yes   | No                                               | No                         |
| Set Profession (Beroepsgroep)                            | Yes   | No, *outside the scope of identity management.*  | No                         |
| VacationRight                                            | Yes   | No, *outside the scope of identity management.*  | No                         |
| AccountAmount                                            | Yes   | No, *outside the scope of identity management.*  | No                         |
| VacationAmounts                                          | Yes   | No, *outside the scope of identity management.*  | No                         |
| CompensationAmount (compensatiesaldo)                    | Yes   | No, *outside the scope of identity management.*  | No                         |
| CompensationSetting (compensatieberekening instellingen) | Yes   | No, *outside the scope of identity management.*  | No                         |
| HourlyWages                                              | Yes   | No, *outside the scope of identity management.*  | No                         |
| CollectiveAgreement                                      | Yes   | No, *outside the scope of identity management.*  | No                         |
| Addresses                                                | Yes   | No, *outside the scope of identity management.*  | No                         |
| FreeField                                                | Yes   | No                                               | No                         |

## Development resources

### API endpoints

The following endpoints are used by the connector

| Endpoint                               | Description                   | Type     |
| -------------------------------------- | ----------------------------- | -------- |
| /t/employees/by_identification_no/<id> | Retrieve Employee Information | Rest     |
| /importws/import                       | CRUD Employee Information     | IOImport |

### API documentation

* Nedap API documentation → [klik](https://www.ons-api.nl/english/technical/APIS.html)
* Nedap XML IOImport Manual → [klik](https://support.nedap-ons.nl/support/solutions/articles/103000265750-gegevens-importeren-via-xml-bestanden)
* Nedap XML IOImport Documentation → [klik](https://hc-freshdesk-assets.s3.eu-central-1.amazonaws.com/Supportportaal%20Ons%20Suite/Technische%20documenten/XML%20Interface/io_server_xml_interface.pdf)
* Nedap ONS authorization Manual → [klik](https://www.ons-api.nl/english/authorization/AuthorizationInOns.html)

## Getting help

> [!TIP]
> _For more information on how to configure a HelloID PowerShell connector, please refer to our [documentation](https://docs.helloid.com/en/provisioning/target-systems/powershell-v2-target-systems.html) pages_.

> [!TIP]
>  _If you need help, feel free to ask questions on our [forum](https://forum.helloid.com/forum/helloid-connectors/provisioning/312-helloid-prov-target-nedap-ons-employee)_.

## HelloID docs

The official HelloID documentation can be found at: https://docs.helloid.com/
