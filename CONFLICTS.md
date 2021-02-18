# CONFLICTS

This document explains why certain conflicts were added to `composer.json` and
references related issues.

 - `symfony/polyfill-mbstring:^1.22.0`:

   `polyfill-mbstring` ^1.22.0 causes a problem with static analysis on PHP 7.3. 
   After running `vendor/bin/psalm --show-info=false --php-version=7.3`, the following exception is thrown:

   `ParseError - vendor/symfony/polyfill-mbstring/bootstrap80.php:125:86 - Syntax error, unexpected '=' on line 125 (see https://psalm.dev/173) function mb_scrub(string $string, string $encoding = null): string { $encoding ??= mb_internal_encoding(); return mb_convert_encoding($string, $encoding, $encoding); }`

   References: https://github.com/vimeo/psalm/issues/4961
