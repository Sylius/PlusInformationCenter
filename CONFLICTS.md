# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `symfony/framework-bundle:6.2.8`:

  This version is missing the service alias `validator.expression`
  which causes ValidatorException exception to be thrown when using `Expression` constraint. 

- `doctrine/orm:2.14.2`:

  This version introduces not configurable XML validation causing errors when using XML mapping files and gedmo extensions.

  Reference: https://github.com/doctrine/DoctrineBundle/issues/1643
