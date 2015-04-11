# LdapTools [![Build Status](https://travis-ci.org/ldaptools/ldaptools.svg)](https://travis-ci.org/ldaptools/ldaptools) [![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/ldaptools/ldaptools/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/ldaptools/ldaptools/?branch=master) [![Latest Stable Version](https://poser.pugx.org/ldaptools/ldaptools/v/stable.svg)](https://packagist.org/packages/ldaptools/ldaptools)
-----------

LdapTools is a feature-rich LDAP library for PHP 5.6+. It was designed to be customizable for use with pretty much any 
directory service, but contains default attribute converters and schemas for Active Directory and OpenLDAP. 
 
 * A fluent and easy to understand syntax for generating LDAP queries.
 * Easily create common LDAP objects (Users, Groups, Contacts, Computers, OUs).
 * Easily modify LDAP objects with automatic setters/getters/properties/etc.
 * Retrieve LDAP objects as either a simple array or an object with automagic setters/getters.
 * A dynamic and customizable attribute converter system to translate data between LDAP and PHP. 
 * Active Directory specific features to help ease development of applications.
 * Includes a comprehensive set of specs for the code.

### Installation

The recommended way to install LdapTools is using [Composer](http://getcomposer.org/download/):

```bash
composer require ldaptools/ldaptools
```

### Getting Started

The easiest way to get started is by creating a YAML config file. See the [example config](resources/config/example.yml) file for basic usage.

Once you have a configuration file defined, you can get up and running by doing the following:

```php
use LdapTools\Configuration;
use LdapTools\LdapManager;

$config = (new Configuration())->load('/path/to/ldap/config.yml');
$ldap = new LdapManager($config);
```

### Searching LDAP

With the `LdapManager` up and going you can now easily build LDAP queries without having to remember all the special 
syntax for LDAP filters. All values are also automatically escaped.

```php
// Get an instance of the query...
$query = $ldap->buildLdapQuery();


// Returns a `LdapObjectCollection` of all users whose first name 
// starts with 'Foo' and last name is 'Bar' or 'Smith'.
// The result set will also be ordered by state name (ascending).
$users = $query->select()
    ->fromUsers()
    ->where($query->filter()->startsWith('firstName', 'Foo'))
    ->orWhere(['lastName' => 'Bar'])
    ->orWhere(['lastName' => 'Smith'])
    ->orderBy('state')
    ->getLdapQuery()
    ->getResult();

echo "Found ".$users->count()." user(s).";
foreach ($users as $user) {
    echo "User: ".$user->getUsername();
}

// Get a single LDAP object...
$user = $ldap->buildLdapQuery()
    ->select()
    ->fromUsers()
    ->where(['username' => 'chad'])
    ->getLdapQuery()
    ->getSingleResult();

// Get a single attribute value from a LDAP object...
$guid = $ldap->buildLdapQuery()
    ->select('guid')
    ->fromUsers()
    ->where(['username' => 'chad'])
    ->getLdapQuery()
    ->getSingleScalarResult();
    
// It also supports the concepts of repositories...
$userRepository = $ldap->getRepository('user');

// Find all users whose last name equals Smith.
$users = $userRepository->findByLastName('Smith');

// Get the first user whose username equals 'jsmith'. Returns a `LdapObject`.
$user = $userRepository->findOneByUsername('jsmith');
echo "First name ".$user->getFirstName()." and last name ".$user->getLastName();
```

The query syntax is very similar to [Doctrine ORM](http://www.doctrine-project.org).

### Modifying LDAP Objects

Modifying LDAP is as easy as searching for the LDAP object as described above, then making changes directly to the object
and saving it back to LDAP using the `LdapManager`.

```php
$user = $ldap->buildLdapQuery()
    ->select(['title', 'mobilePhone', 'disabled'])
    ->fromUsers()
    ->where(['username' => 'jsmith'])
    ->getLdapQuery()
    ->getSingleResult();

// Make some modifications to the user account.
All these changes are tracked so it knows how to modify the object.
$user->setTitle('CEO');

if ($user->hasMobilePhone()) {
    $user->resetMobilePhone();
}

// Set a field by a property instead...
if ($user->disabled) {
    $user->disabled = false;
}

// Now actually save the changes back to LDAP...
try {
    $ldap->persist($user);
} catch (\Exception $e) {
    echo "Error updating user! ".$e->getMessage();
}
```

### Deleting LDAP Objects

Deleting LDAP objects is a simple matter of searching for the object you want to remove, then passing it to the delete
method on the `LdapManager`:

```php
// Decide they no longer work here and should be deleted?
$user = $userRepository->findOneByUsername('jsmith');

try {
    $ldap->delete($user);
} catch (\Exception $e) {
    echo "Error deleting user! ".$e->getMessage();
}
```

### Creating LDAP Objects
 
Creating LDAP objects is easily performed by just passing what you want the attributes to be and what container/OU the
object should end up in:

```php
$ldapObject = $ldap->createLdapObject();

// Creating a user account (enabled by default)
$ldapObject->createUser()
    ->in('cn=Users,dc=example,dc=local')
    ->with(['username' => 'jsmith', 'password' => '12345'])
    ->execute();

// Create a typical AD global security group...
$ldapObject->createGroup()
    ->in('dc=example,dc=local')
    ->with(['name' => 'Generic Security Group'])
    ->execute();

// Creates a contact user...
$ldapObject->createContact()
    ->in('dc=example,dc=local')
    ->with(['name' => 'Some Guy', 'emailAddress' => 'SomeGuy@SomeDomain.com'])
    ->execute();

// Creates a computer object...
$ldapObject->createComputer()
    ->in('dc=example,dc=local')
    ->with(['name' => 'MYWOKRSTATION'])
    ->execute();
    
// Creates an OU object...
$ldapObject->createOU()
    ->in('dc=example,dc=local')
    ->with(['name' => 'Employees'])
    ->execute();
```

### Documentation

Browse [the docs folder](/docs/en) for more information about LdapTools.

* [Main Configuration Reference](/docs/en/reference/Main-Configuration.md)
* [Schema Configuration](/docs/en/reference/Schema-Configuration.md)
* [Using the LdapManager](/docs/en/tutorials/Using-the-LDAP-Manager.md)
* [Building LDAP Queries](/docs/en/tutorials/Building-LDAP-Queries.md)
* [Creating LDAP Objects](/docs/en/tutorials/Creating-LDAP-Objects.md)
* [Modifying LDAP Objects](/docs/en/tutorials/Modifying-LDAP-Objects.md)
* [Default Schema Attributes](/docs/en/reference/Default-Schema-Attributes.md)

### TODO

Things that still need to be implemented:

* Automatic generation of the schema based off of information in LDAP.
* A logging mechanism.
* An event system.
* More work needed on the OpenLDAP schema.
