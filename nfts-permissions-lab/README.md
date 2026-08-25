# NTFS Permissions Lab

## Overview

This lab demonstrates practical Windows file-system administration and Help Desk troubleshooting involving **NTFS permissions**.

The objective was to configure and test access to a Windows folder by assigning appropriate permissions to users and verifying how Windows handles authorized and unauthorized access.

The lab was performed in a cloud-based Windows environment and documented using screenshots and command-line verification.

## Objectives

* Create and configure a test directory for permission management.
* Understand NTFS permission inheritance.
* Assign permissions to users.
* Test authorized and unauthorized access.
* Verify effective access using Windows tools and commands.
* Troubleshoot permission-related access issues.
* Document the configuration and resolution in a Help Desk format.

## Environment

| Component        | Configuration                                  |
| ---------------- | ---------------------------------------------- |
| Platform         | AWS EC2                                        |
| Operating System | Windows Server                                 |
| File System      | NTFS                                           |
| Access Method    | Remote Desktop                                 |
| Management Tools | Windows Explorer / PowerShell / Command Prompt |
| Lab Type         | IT Support / Windows Administration            |

## Scenario

A user reported that they were unable to access a folder required for their work.

The Help Desk investigation required determining whether the problem was caused by incorrect NTFS permissions, inheritance, or the user's assigned access.

The objective was to configure the folder so that authorized users could access the required resources while unauthorized users remained restricted.

## Lab Procedure

### 1. Prepare the Windows Environment

The Windows system was accessed through Remote Desktop.

The test environment was verified before beginning the permissions configuration.

**Evidence:**
See the corresponding screenshots in the `screenshots/` directory.

### 2. Create the Test Folder

A dedicated test folder was created for the permissions exercise.

The folder served as the controlled location for testing NTFS access.

### 3. Configure NTFS Permissions

The folder's security properties were opened and the required user/group permissions were configured.

NTFS permissions determine which users can perform operations such as:

* Read files
* Write files
* Modify files
* Delete files
* Create files and folders
* Read folder contents

The permissions were configured according to the access requirements of the scenario.

### 4. Verify Permission Assignment

The assigned permissions were reviewed through the folder's **Security** properties.

This confirmed which users and groups had access and what level of access had been granted.

### 5. Test Authorized Access

The configured user was used to verify that the expected operations were permitted.

Successful access confirmed that the NTFS permissions were functioning as intended.

### 6. Test Restricted Access

Access was also tested from a user without the required permissions.

The purpose of this test was to verify that Windows correctly denied unauthorized access.

This is an important part of permissions troubleshooting because simply assigning permissions is not enough; the resulting access behavior must also be verified.

## Troubleshooting Approach

The permissions issue was approached systematically:

1. Identify the affected folder.
2. Determine which user was experiencing the problem.
3. Inspect the folder's Security properties.
4. Review assigned users and groups.
5. Check the permission level assigned to the affected account.
6. Consider inherited permissions.
7. Modify the permissions where required.
8. Retest access.
9. Confirm that the final behavior matched the intended access policy.

## NTFS Permissions Concepts Demonstrated

### Read

Allows a user to view files and folder contents.

### Write

Allows a user to create or write data within the permitted location.

### Modify

Provides broader access, including modifying and deleting files.

### Full Control

Provides complete control over the object, including changing permissions.

### Permission Inheritance

NTFS permissions can be inherited from parent directories.

This means that permissions assigned at a higher level can automatically apply to child folders and files unless inheritance is disabled or permissions are explicitly configured differently.

## Verification

The final configuration was verified by testing the expected access behavior.

The screenshots provide evidence of:

* The Windows environment
* Folder configuration
* Security/permissions configuration
* Permission verification
* Access testing
* Final state of the lab

## Skills Demonstrated

* Windows Server administration
* NTFS permissions
* File-system access control
* Permission troubleshooting
* User access management
* Windows security configuration
* Remote Desktop administration
* Command-line verification
* Technical documentation
* Help Desk troubleshooting methodology

## Conclusion

This lab demonstrated how NTFS permissions can be used to control access to files and folders in a Windows environment.

The exercise reinforced the importance of checking the actual permission configuration rather than assuming that an access problem is caused by the user's account or the application.

From a Help Desk perspective, the same troubleshooting process can be applied to real-world tickets involving users who cannot open, modify, create, or delete files in shared or local Windows directories.
