# USMT

## Introduction

USMT (User State Migration Tool) is a Microsoft command-line utility for migrating user profiles, files, and Windows settings between computers or operating system installations. Included in the Windows Assessment and Deployment Kit (Windows ADK), USMT supports enterprise Windows 10 and Windows 11 deployment, device replacement, and operating system refresh projects. It captures selected user data from a source computer and restores that data to a destination without transferring the entire operating system.

The migration workflow uses three primary executables. ScanState.exe collects user state and writes it to a migration store. LoadState.exe restores the captured files and settings. USMTUtils.exe provides migration-store maintenance and verification capabilities. Administrators control the scope of migration through XML rules rather than manually copying individual profile directories.

MigDocs.xml identifies user documents through file-location and pattern-based discovery, while MigApp.xml defines supported application settings. Custom XML files can include or exclude specific folders, file types, and registry settings. Config.xml can disable selected migration components. USMT can migrate multiple user accounts, including domain profiles, and offers account-selection and mapping options.

Typical use cases include replacing employee laptops, preserving user environments during managed Windows upgrades, and integrating profile migration into automated deployment sequences. USMT does not install applications or create a complete disk image; required software must be deployed separately. Because migration depends on supported settings, profile selection, and storage permissions, administrators should test the configuration and maintain an independent backup before production deployment.

## How to Use USMT ScanState and LoadState for Windows User Profile Migration

Install the Windows ADK with the User State Migration Tool feature on an administrative workstation or deployment environment. Use the appropriate USMT architecture, ensure sufficient space in the destination store, and run migration commands from an elevated Command Prompt. The source account must have access to the files being collected; a network store also requires appropriate share and NTFS permissions.

For a standard Windows user profile migration, capture user documents and supported application settings with:

```
ScanState.exe \\server\migration\PC01 /i:MigDocs.xml /i:MigApp.xml /v:5 /l:C:\Logs\ScanState.log
```

Run the command from the USMT directory, or provide full paths to its executables and XML files. The specified store receives captured data; the log file must be stored separately. Close applications before capture to reduce locked-file errors.

After installing Windows and required applications on the destination PC, restore the user state:

```
LoadState.exe \\server\migration\PC01 /i:MigDocs.xml /i:MigApp.xml /v:5 /l:C:\Logs\LoadState.log
```

Use the same migration XML rules for both stages. For domain profiles, join the destination to the domain before users need to access their restored profiles. Sign out and back in to apply settings that initialize at logon.

Check command exit codes and migration logs before retiring the original device. Validate desktop files, user documents, application preferences, and account access with the actual user. USMT migration is not a substitute for a tested backup and recovery plan.

## USMT Configuration: Migration XML Files, User Account Filters, and Troubleshooting

USMT configuration determines which data is captured, which accounts are processed, and how failures are handled. For more precise control than default document discovery, add a custom migration XML file with `/i:Custom.xml` to both ScanState and LoadState. Keep original built-in XML files unchanged and validate custom rules on representative computers.

To generate a component configuration file, run:

```
ScanState.exe /genconfig:Config.xml /i:MigDocs.xml /i:MigApp.xml
```

Edit the generated Config.xml and set `migrate="no"` for components that should be excluded. Pass `/config:Config.xml` when capturing the store; use it during restoration if exclusions also need to be enforced at that stage. Component exclusions and file-selection rules serve different purposes: Config.xml switches components on or off, while migration XML controls individual file and registry selection.

Account filters help prevent obsolete profiles from being transferred. For example, `/ue:*\* /ui:CONTOSO\jdoe` restricts collection to a specified domain user. The `/uel:90` option can limit collection to profiles with recent activity, but should be tested against enterprise account-retention requirements.

For troubleshooting, `/l:` creates a diagnostic log, `/progress:` records migration progress, and `/v:5` increases detail. The `/c` switch permits execution to continue after nonfatal errors; review all skipped items rather than treating completion as full success. For migrations involving sensitive user data, secure the network share and consider USMT store encryption. Avoid embedding encryption keys directly in broadly accessible deployment scripts.
