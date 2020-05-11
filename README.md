<p align="center">
    <a href="https://sylius.com" target="_blank">
        <img src="https://demo.sylius.com/assets/shop/img/logo.png" />
    </a>
</p>

<h1 align="center">Sylius Plus</h1>

<p align="center">A set of powerful extensions for enterprise customers.</p>

## Installation for users

[Sylius Documentation > The Book > Installation > Sylius Plus Installation](https://docs.sylius.com/en/1.7/book/installation/sylius_plus_installation.html)

## Documentation

Sylius Plus documentation can be found on [docs.sylius.com](https://docs.sylius.com/en/1.7/).

## Testing

### Running tests

- PHPUnit

    ```bash
    $ vendor/bin/phpunit
    ```

- PHPSpec

    ```bash
    $ vendor/bin/phpspec run
    ```

- Behat (non-JS scenarios)

    ```bash
    $ vendor/bin/behat --tags="~@javascript"
    ```

- Behat (JS scenarios)

1. Download [Chromedriver](https://sites.google.com/a/chromium.org/chromedriver/)

2. Download [Selenium Standalone Server](https://www.seleniumhq.org/download/).

2. Run Selenium server with previously downloaded Chromedriver:

    ```bash
    $ java -Dwebdriver.chrome.driver=chromedriver -jar selenium-server-standalone.jar
    ```
    
3. Run test application's webserver on `localhost:8080`:

    ```bash
    $ (cd tests/Application && APP_ENV=test bin/console server:run localhost:8080 -d public)
    ```

4. Run Behat:

    ```bash
    $ vendor/bin/behat --tags="@javascript"
    ```

### Runing Plus in your browser

- Using `test` environment:

    ```bash
    $ (cd tests/Application && APP_ENV=test bin/console sylius:fixtures:load plus)
    $ (cd tests/Application && APP_ENV=test bin/console server:run -d public)
    ```
    
- Using `dev` environment:

    ```bash
    $ (cd tests/Application && APP_ENV=dev bin/console sylius:fixtures:load plus)
    $ (cd tests/Application && APP_ENV=dev bin/console server:run -d public)
    ```
