# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

- `symfony/framework-bundle:4.4.38|5.4.5`:
  
  These versions are causing a problem with mink session: 
  `InvalidArgumentException: Specify session name to get in vendor/friends-of-behat/mink/src/Mink.php:198`
