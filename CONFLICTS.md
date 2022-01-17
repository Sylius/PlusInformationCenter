# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `doctrine/orm:^2.10.0`:

  This version causes a problem with the creation of nested taxons by throwing the exception:

  `Gedmo\Exception\UnexpectedValueException: Root cannot be changed manually, change parent instead in vendor/gedmo/doctrine-extensions/src/Tree/Strategy/ORM/Nested.php:145`

  References: https://github.com/doctrine-extensions/DoctrineExtensions/issues/2155
