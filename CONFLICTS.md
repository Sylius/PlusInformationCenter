# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `symfony/framework-bundle:6.2.8`:

  This version is missing the service alias `validator.expression`
  which causes ValidatorException exception to be thrown when using `Expression` constraint. 

- `behat/mink-selenium2-driver:>=1.7.0`:

  This version adds strict type to the `Behat\Mink\Driver\Selenium2Driver::visit($url)` method
  which causes a fatal error because the method signature is no longer compatible with the parent class.
