# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `doctrine/orm:^2.10.0`:

  This version causes a problem with the creation of nested taxons by throwing the exception:

  `Gedmo\Exception\UnexpectedValueException: Root cannot be changed manually, change parent instead in vendor/gedmo/doctrine-extensions/src/Tree/Strategy/ORM/Nested.php:145`

  References: https://github.com/doctrine-extensions/DoctrineExtensions/issues/2155

- `payum/payum:^1.7.0`:

  This version causes a problem with the database schema out of sync with the current mapping file after migrations.
  The conflict should be removed after upgrading to Sylius 1.10.8.

  `[ERROR] The database schema is not in sync with the current mapping file.`

  References: https://github.com/Sylius/Sylius/pull/13214
