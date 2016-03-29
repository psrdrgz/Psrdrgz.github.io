---
layout: post
title: Working With Active Directory - Computers
---

**Scenario:** Person A has a script that they found to do X against a list of server names. However, it seems that it only works if the names of the server are pulled in via text file. Person A wants to be able to pull the names of the servers via Active Directory.

Luckily for Person A, PowerShell can query Active Directory (AD) very, very easily.

Now, there are two primary methods of querying AD with PowerShell: the AD cmdlets available from Remote Server Administration Tools ([RSAT](https://support.microsoft.com/en-us/kb/2693643)) or the Active Directory Service Interfaces ([ADSI](https://msdn.microsoft.com/en-us/library/windows/desktop/aa772170(v=vs.85).aspx)).

Without a doubt the easiest way of interacting with AD is via the AD cmdlets. How can you tell if they're installed? Why not use PowerShell and check to see if RSAT is installed?

```powershell
Get-WmiObject -Class Win32_QuickFixEngineering -Filter "HotFixID='KB2693643'"
```

If the above command does not return any results, then I'm afraid you more than likely don't have RSAT installed. But just because it's not installed doesn't mean you can't boogey along. Don't forget about ADSI. To the best of my knowledge Windows OS's from XP to 10 all can make use of ADSI to varying degrees.

So without any further adieu here's how we can use PowerShell to retrive the names of all the computers in Active Directory.

*To list the names of all the computers in the domain:*

```powershell
# Using the AD cmdlets
(Get-ADComputer -Filter *).Name

# Using the ADSISearcher PowerShell Type Accelerator
([adsisearcher]"objectclass=computer").FindAll().Properties.name

```

But what if you want to only list the computers in a particular organizational unit (OU)? No problemo. Just so long as you know the distinguished name of the OU.

Don't know the distinguished name or afraid you'll mistype it? No problemo.

*To find the distinguished name of an OU from its name:*

```powershell
# Using the AD cmdlets
(Get-ADOrganizationalUnit -Filter "Name -like '*Servers*'").DistinguishedName

# Using the ADSISearcher PowerShell Type Accelerator
([adsisearcher]"(&(objectclass=organizationalunit)(name=*Servers*))").FindAll().Properties.distinguishedname

# Note: To make use of the above commands simply replace Servers with with the name of your OU.
```

*To list the names of all the computers in a particular OU:*

```powershell
# Using the AD cmdlets
(Get-ADComputer -Filter * -SearchBase "OU=Servers,DC=contoso,DC=com").Name

# Using the DirectoryServices DirectorySearcher class, the class which the ADSISearcher PowerShell Type accelerator abstracts
(New-Object -TypeName adsisearcher -ArgumentList ([adsi]'LDAP://OU=Servers,DC=contoso,DC=com', '(objectclass=computer)')).FindAll().Properties.name

# Note: To make use of the above commands simply replace OU=Servers,DC=contoso,DC=com with with the distinguished name of your OU.
```

Pretty simple, huh? 

Well let me tell you my friend, we haven't even scratched the surface. 

