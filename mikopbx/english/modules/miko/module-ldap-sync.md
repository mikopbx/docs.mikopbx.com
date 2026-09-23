# Synchronization with LDAP/AD

This module is designed for bidirectional synchronization of employee account data with MikoPBX. The data source is either an Active Directory or LDAP server.

When a new employee is created in the domain, they will automatically be uploaded into MikoPBX, with an available internal number assigned to them. The information about the number will then be sent back to the domain and recorded in the employee's profile. The same will happen with their mobile phone number and photograph. As always, the setup is extremely straightforward.

<figure><img src="../../.gitbook/assets/ModuleLdapSync - index.png" alt=""><figcaption><p>Setting up Active Directory servers for user synchronization with MikoPBX</p></figcaption></figure>

## Synchronization Parameters

For each AD/LDAP server, you can specify synchronization settings, departmental filters, or create a custom filter for complex filtering logic.

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 1.png" alt=""><figcaption><p>Domain controller synchronization settings</p></figcaption></figure>

## Account Attributes

Next, you need to correctly configure the attributes for synchronizing account data.

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 2.png" alt=""><figcaption><p>Setting up synchronization attributes between MikoPBX and the domain</p></figcaption></figure>

## Initial Synchronization

During the initial synchronization, the system will match the existing MikoPBX account data with the data obtained from the domain. The following fields are used for matching:

* Email address
* Employee name
* Mobile phone
* Internal phone

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 4.png" alt=""><figcaption><p>Testing data synchronization with AD/LDAP</p></figcaption></figure>

## Testing and Launch

Before enabling automatic synchronization, it is recommended to test the correctness of the specified attributes by clicking the **Run request** button.

If all parameters are correctly specified, you will see a list of employees with attributes from the domain. This is a safe request and will not result in changes to the system.

## Next Steps

After testing, you can initiate manual or automatic data synchronization.

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 5.png" alt=""><figcaption><p>Status of employee synchronization between the domain and MikoPBX</p></figcaption></figure>

In the columns "_status_" and "_updated_", you can track the current synchronization process.

### Employee Deletion or Deactivation in the Domain

When an employee is deleted or deactivated in the domain, they will remain active in MikoPBX but will be moved to a special table called “**Disabled Employees in LDAP/AD**.”

The account will be retained until it is manually deleted by the MikoPBX administrator.

This approach accommodates complex call routing scenarios where simply removing the employee from the route without a replacement is not feasible.

### Synchronization Conflicts

During synchronization, conflicts may occur if the system fails to create or modify employee data in MikoPBX or on the LDAP/AD server. All synchronization issues are logged by the module and recorded in a special table titled “**Synchronization Conflicts**.”

The MikoPBX administrator can manually resolve these issues and clear the conflicts table in the module.

### SIP Password Synchronization

The module includes an option for synchronizing SIP passwords with the domain controller. This can be useful for automatically configuring IP phones within the company based on domain data. To enable synchronization, a special attribute must be created on the domain controller side, and this attribute should be specified in the “**SIP Password**” field in the attribute mapping settings for synchronization.

Note that the password will be stored in plain text. With bidirectional synchronization enabled, the password value from MikoPBX will be sent to the domain and vice versa, based on the date of the last modification.

Please note: The SIP password is not the same as the domain account password; it is a separate value.
