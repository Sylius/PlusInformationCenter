# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `sylius/refund-plugin:1.4.0`:

  This version introduced changes breaking fixtures loading and static analysis.

- `symfony/framework-bundle:6.2.8`:

  This version is missing the service alias `validator.expression`
  which causes ValidatorException exception to be thrown when using `Expression` constraint. 
