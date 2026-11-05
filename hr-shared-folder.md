INC007 --- HR Shared Folder: Unable to Save Changes
Scenario: Simulated Help Desk incident
Status: Troubleshooting procedure documented; no live investigation
or resolution has been verified.
Symptoms
An HR employee can open the HR shared folder and view documents, but
receives a permissions error when attempting to save changes to an
existing document.
Investigation
The following checks should be completed using the affected employee's
account and the relevant HR security groups.
1. Check SMB share permissions
   - Confirm that the employee's intended HR security group is
     granted Change access (or an appropriately broader
     permission) on the share.
   - Read-only share access can allow the employee to open documents
     while preventing changes over the network.
2. Check NTFS permissions
   - Inspect the permissions on the shared folder and the affected
     document.
   - Confirm that the intended HR group has Modify permission
     where editing is required.
   - Read and Read & Execute alone do not provide the normal
     permissions needed to edit existing files.
3. Verify group membership
   - Confirm that the employee is a member of the correct HR security
     group.
   - If membership was recently changed, ensure the employee has a
     refreshed sign-in session/token before retesting.
4. Inspect explicit Deny entries and inheritance
   - Check for applicable Deny permissions.
   - Confirm that permissions are inherited as intended and that a
     child folder or file does not have unexpected permission
     overrides.
Findings
Not yet established. This is a simulated scenario and the checks
above have not been performed against a live system. Do not record a
specific SMB, NTFS, group-membership, Deny, or inheritance fault as the
cause until evidence confirms it.
Resolution
Pending investigation. Make only the permission change supported by
the findings and consistent with the organization's access policy. Avoid
granting broader access than the employee needs.
Verification
After any approved change:
1. Sign in as the affected employee, or use an appropriate test account
   with the same group membership.
2. Access the HR folder through its UNC network path.
3. Open a test document and attempt to save an edit.
4. Confirm that the employee can perform the intended tasks and cannot
   perform actions outside the approved access level.
5. Record the test result and any remaining limitations.
Verification status: Not performed; no successful resolution is
claimed.
