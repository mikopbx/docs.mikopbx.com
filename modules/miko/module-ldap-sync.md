# Synchronization with LDAP/AD

This module is designed for bidirectional synchronization of employee account data with MikoPBX. The data source is either an Active Directory or LDAP server.

<figure><img src="../../.gitbook/assets/ModuleLdapSync%20-%20index.png" alt=""><figcaption><p>Setting up Active Directory servers for user synchronization with MikoPBX</p></figcaption></figure>

## Synchronization Parameters

For each AD/LDAP server, you can specify synchronization settings, departmental filters, or create a custom filter for complex filtering logic.

<figure><img src="../../.gitbook/assets/ModuleLdapSync%20-%20modify%201.png" alt=""><figcaption><p>Domain controller synchronization settings</p></figcaption></figure>

## Account Attributes

Next, you need to correctly configure the attributes for synchronizing account data.

<figure><img src="../../.gitbook/assets/ModuleLdapSync%20-%20modify%202.png" alt=""><figcaption><p>Setting up synchronization attributes between MikoPBX and the domain</p></figcaption></figure>

## Initial Synchronization

During the initial synchronization, the system will match the existing MikoPBX account data with the data obtained from the domain. The following fields are used for matching:

* Email address
* Employee name
* Mobile phone
* Internal phone

<figure><img src="../../.gitbook/assets/ModuleLdapSync%20-%20modify%204.png" alt=""><figcaption><p>Testing data synchronization with AD/LDAP</p></figcaption></figure>

## Testing and Launch

Before enabling automatic synchronization, it is recommended to test the correctness of the specified attributes by clicking the **Execute Request** button.

If all parameters are correctly specified, you will see a list of employees with attributes from the domain. This is a safe request and will not result in changes to the system.

## Next Steps

After testing, you can initiate manual or automatic data synchronization.

<figure><img src="../../.gitbook/assets/ModuleLdapSync%20-%20modify%205.png" alt=""><figcaption><p>Status of employee synchronization between the domain and MikoPBX</p></figcaption></figure>

In the columns "_status_" and "_updated_", you can track the current synchronization process.

<div>

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 1.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 2.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 3.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/ModuleLdapSync - index.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 4.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../../.gitbook/assets/ModuleLdapSync - modify 5.png" alt=""><figcaption></figcaption></figure>

</div>

{% hint style="info" %}
As of now, the removal of an employee from the domain will not lead to their automatic removal from MikoPBX. The account will be retained until manually deleted by a MikoPBX administrator. This is because there may be various complex call routing scenarios where it's not feasible to simply remove an employee from the call route without replacing them with someone else.
{% endhint %}
