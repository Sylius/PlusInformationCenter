# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `doctrine/orm:^2.10.0`:

  This version causes a problem with the creation of nested taxons by throwing the exception:

  `Gedmo\Exception\UnexpectedValueException: Root cannot be changed manually, change parent instead in vendor/gedmo/doctrine-extensions/src/Tree/Strategy/ORM/Nested.php:145`

  References: https://github.com/doctrine-extensions/DoctrineExtensions/issues/2155

- `symfony/framework-bundle:4.4.38|5.4.5`:
  
  These versions are causing a problem with mink session: 
  `InvalidArgumentException: Specify session name to get in vendor/friends-of-behat/mink/src/Mink.php:198`
