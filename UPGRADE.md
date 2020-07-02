# UPGRADE FROM 0.22.0 to 0.23.0

## General update

```bash
composer require "sylius/plus:0.23.*"
```

* `BusinessUnitFixture` service has been moved and changed id from `sylius.fixture.business_unit` to `Sylius\Plus\BusinessUnits\Infrastructure\Fixture\BusinessUnitFixture`

* Some fixtures services have been moved to proper namespaces and changed their ids:
    * `Sylius\Plus\Fixture\Factory\BusinessUnitExampleFactory` to `Sylius\Plus\BusinessUnits\Infrastructure\Fixture\Factory\BusinessUnitExampleFactory`
    * `Sylius\Plus\Fixture\Factory\BusinessUnitAddressExampleFactory` to `Sylius\Plus\BusinessUnits\Infrastructure\Fixture\Factory\BusinessUnitAddressExampleFactory`
    * `Sylius\Plus\Fixture\Factory\InventorySourceExampleFactory` to `Sylius\Plus\Inventory\Infrastructure\Fixture\Factory\InventorySourceExampleFactory`
    * `Sylius\Plus\Fixture\InventorySourceFixture` to `Sylius\Plus\Inventory\Infrastructure\Fixture\InventorySourceFixture`
    * `Sylius\Plus\Fixture\InventorySourceStockFixture` to `Sylius\Plus\Inventory\Infrastructure\Fixture\InventorySourceStockFixture`
    * `Sylius\Plus\Fixture\ReturnRequestFixture` to `Sylius\Plus\Returns\Infrastructure\Fixture\ReturnRequestFixture`

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20200603132754.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200604060510.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200605081658.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200605143716.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200610131835.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200624071028.php src/Migrations/
```

# UPGRADE FROM 0.21.0 to 0.22.0

## General update

```bash
composer require "sylius/plus:0.22.*"
```

## Update templates

* To fix the templates overwriting problem and to be more consistent with Symfony Framework, all templates from other localization than `src/Resources/views/*` were moved there.

  For example, if one overridden `SyliusPlusPlugin/Inventory/Infrastructure/Resources/views/` templates then files from `templates/bundles/SyliusPlusPlugin/Inventory/Infrastructure/Resources/views/` should be moved to `templates/bundles/SyliusPlusPlugin/Resources/Inventory/`, 
  you can use commands to make it easier:
 
  ```bash
  mv templates/bundles/SyliusPlusPlugin/BusinessUnits/Infrastructure/Resources/views/ templates/bundles/SyliusPlusPlugin/Resources/BusinessUnits/
  mv templates/bundles/SyliusPlusPlugin/CustomerPools/Infrastructure/Resources/views/ templates/bundles/SyliusPlusPlugin/Resources/CustomerPools/
  mv templates/bundles/SyliusPlusPlugin/Inventory/Infrastructure/Resources/views/ templates/bundles/SyliusPlusPlugin/Resources/Inventory/
  mv templates/bundles/SyliusPlusPlugin/Rbac/Infrastructure/Resources/views/ templates/bundles/SyliusPlusPlugin/Resources/Rbac/
  mv templates/bundles/SyliusPlusPlugin/Returns/Infrastructure/Resources/views/ templates/bundles/SyliusPlusPlugin/Resources/Returns/
  ```
 
  If you've imported or extended any of these templates you must change paths as below:
 
    * `@SyliusPlusPlugin/BusinessUnits/Infrastructure/Resources/views/*` to `@SyliusPlusPlugin/BusinessUnits/*`
    * `@SyliusPlusPlugin/CustomerPools/Infrastructure/Resources/views/*` to `@SyliusPlusPlugin/CustomerPools/*`
    * `@SyliusPlusPlugin/Inventory/Infrastructure/Resources/views/*` to `@SyliusPlusPlugin/Inventory/*`
    * `@SyliusPlusPlugin/Rbac/Infrastructure/Resources/views/*` to `@SyliusPlusPlugin/Rbac/*`
    * `@SyliusPlusPlugin/Returns/Infrastructure/Resources/views/*` to `@SyliusPlusPlugin/Returns/*`

* Update file `templates/bundles/SyliusAdminBundle/Product/Show/_variantContent.html.twig` 
 by removing duplicated shipping section:

    ```diff
        ...
        <div class="doubling two column row">
            <div class="column">
                {% include '@SyliusPlusPlugin/Admin/Product/Show/_inventory.html.twig' %}
    -           {% include '@SyliusAdmin/Product/Show/_variantContentShipping.html.twig' %}
            </div>
    ```

# UPGRADE FROM 0.20.0 to 0.21.0

## General update

```bash
composer require "sylius/plus:0.21.*"
```

* `Sylius\Plus\Doctrine\ORM\OrderRepository` and `Sylius\Plus\Doctrine\ORM\OrderRepositoryInterface` have been removed.

## Update templates

* Copy new templates that are overriden by Sylius Plus into `templates/bundles`:
    ```
    vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Shipment/Show/_breadcrumb.html.twig
    vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Shipment/show.html.twig
    ```

* Update file `templates/bundles/SyliusAdminBundle/Order/Show/_shipment.html.twig` by adding:
 
    ```diff
        ...
        {% set shipped = constant('Sylius\\Component\\Shipping\\Model\\Shipment::STATE_SHIPPED') %}
    +   {% set cart = constant('Sylius\\Component\\Shipping\\Model\\Shipment::STATE_CART') %}
        ...
        {% endif %}
    -   {% if shipment.state == shipped %}
    +   {% if shipment.state != cart %}
            <a class="ui icon labeled tiny fluid button" style="color: gray; margin-top: 10px;" href="{{ path('sylius_admin_shipment_show', {'id': shipment.id}) }}" {{ sylius_test_html_attribute('shipment-show-button') }}>
                <i class="icon search"></i>{{ 'sylius.ui.show'|trans }}
            </a>
    ```

# UPGRADE FROM 0.19.0 to 0.20.0

## General update

```bash
composer require "sylius/plus:0.20.*"
```

* Inventory stocks before 0.20 were reduced for all units in the order after payment was completed.
 Since 0.20 they are reduced after the shipment is shipped and only for the shipped units.
 Since now inventory stock will be reduced after shipping, not after payment.

* If you've customised the logic related to reducing inventory stock in `Sylius\Plus\Inventory\Application\Operator\SellOrderInventoryOperator`,
 move your changes to `Sylius\Plus\Inventory\Application\Operator\ShipShipmentInventoryOperator`. 
 Keep in mind that the new inventory operator runs when a shipment is shipped, as opposed to the old one which runs when payment for an order is completed.
 
## Update templates

* Update file `templates/bundles/SyliusAdminBundle/Order/Show/_shipment.html.twig` by adding:
 
    ```diff
        ...
        {% if shipment.tracking is not empty %}
            <div class="description">
                <i class="barcode icon"></i><span>{{ shipment.tracking }}</span>
            </div>
        {% endif %}
    +   {% if shipment.state == shipped %}
    +       <a class="ui icon labeled tiny fluid button" style="color: gray; margin-top: 10px;" href="{{ path('sylius_admin_shipment_show', {'id': shipment.id}) }}" {{ sylius_test_html_attribute('shipment-show-button') }}>
    +           <i class="icon search"></i>{{ 'sylius.ui.show'|trans }}
    +       </a>
    +   {% endif %}
        ...
                <i class="barcode icon"></i><span>{{ shipment.tracking }}</span>
            </div>
        {% endif %}
    +   {% if shipment.state == shipped %}
    +       <div class="ui icon labeled tiny fluid button" style="margin-top: 10px;">
    +           <i class="icon search"></i><a style="color: gray;" href="{{ path('sylius_admin_shipment_show', {'id': shipment.id}) }}" {{ sylius_test_html_attribute('shipment-show-button') }}>{{ 'sylius.ui.show'|trans }}</a>
    +           </div>
    +    {% endif %}
        ...
    ```
    
* Update file `templates/bundles/SyliusAdminBundle/Channel/_form.html.twig` by adding:

    ```diff
        ...
        <div class="ui attached segment">
            {{ form_row(form.locales) }}
            {{ form_row(form.defaultLocale) }}
        </div>
    +   <div class="ui attached segment">
    +       {{ form_row(form.menuTaxon) }}
    +   </div>
        <div class="ui hidden divider"></div>
        ...
    ```

# UPGRADE FROM 0.18.0 to 0.19.0

## General update

```bash
composer require "sylius/plus:0.19.*"
```

## Update templates

```bash
cp vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Dashboard/_channelSwitch.html.twig templates/bundles/SyliusAdminBundle/Dashboard/_channelSwitch.html.twig
```

# UPGRADE FORM 0.17.0 0.18.0

## General update

```bash
composer require "sylius/plus:0.18.*"
```

## Update templates

```bash
cp vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Order/Show/_shipments.html.twig templates/bundles/SyliusAdminBundle/Order/Show/_shipments.html.twig
```

* Update file `templates/bundles/SyliusAdminBundle/Order/Show/_shipment.html.twig` by adding:
 
    ```diff
        ...
            </div>
    -       <i class="large truck icon"></i>
            <div class="content">
                <div class="header">
    +               <i class="large truck icon"></i>
                    {{ shipment.method }}
        ...
                {% if shipment.shippedAt is not empty %}
    -               {{ 'sylius.ui.shipped_at'|trans }}: <span class="shipped-at-date">{{ shipment.shippedAt|date('d-m-Y H:i:s') }}</span>
    +               <div class="description">
    +                   <i class="calendar alternate icon"></i> <span class="shipped-at-date">{{ shipment.shippedAt|format_datetime }}</span>
    +               </div>
    +           {% endif %}
    +           {% if shipment.tracking is not empty %}
    +               <div class="description">
    +                   <i class="barcode icon"></i><span>{{ shipment.tracking }}</span>
    +               </div>
                {% endif %}
            </div>
            {% if sm_can(shipment, 'ship', 'sylius_shipment') %}
                {{ render(path('sylius_admin_partial_shipment_ship', {'orderId': order.id, 'id': shipment.id})) }}
            {% endif %}
    -       {% if shipment.tracking is not empty %}
    -           <div class="ui segment">
    -               <span class="ui top attached icon label"><i class="plane icon"></i> {{ 'sylius.ui.tracking_code'|trans|upper }}</span>
    -               <p>{{ shipment.tracking }}</p>
    -           </div>
    -       {% endif %}
        
            {% if shipment.state == shipped %}
    -           {% set path = path('sylius_admin_shipment_resend_confirmation_email', {'id': shipment.id, '_csrf_token': csrf_token(shipment.id)}) %}
    -           <a href="{{ path }}" class="ui icon labeled tiny fluid button" style="margin: 7px 0 0;" {{ sylius_test_html_attribute('resend-shipment-confirmation-email') }}>
    -               <i class="send icon"></i> {{ 'sylius.ui.resend_the_shipment_confirmation_email'|trans }}
    -           </a>
    +           <div style="margin-bottom: 10px;">
    +               {% set path = path('sylius_admin_shipment_resend_confirmation_email', {'id': shipment.id, '_csrf_token': csrf_token(shipment.id)}) %}
    +               <a href="{{ path }}" class="ui icon labeled tiny fluid button" style="margin: 7px 0 0;" {{ sylius_test_html_attribute('resend-shipment-confirmation-email') }}>
    +                   <i class="send icon"></i> {{ 'sylius.ui.resend_the_shipment_confirmation_email'|trans }}
    +               </a>
    +           </div>
            {% endif %}
        </div>
        ...
    ```

# UPGRADE FORM 0.16.0 0.17.0

## General update

```bash
composer require "sylius/plus:0.17.*"
```

* You need to upgrade Sylius to version 1.7 and add necessary migrations: https://github.com/Sylius/Sylius/blob/master/UPGRADE-1.7.md

## Update templates

* Update file `templates/bundles/SyliusAdminBundle/Order/Show/_shipment.html.twig` by adding:

    ```diff
        ...
    +   {% set shipped = constant('Sylius\\Component\\Shipping\\Model\\Shipment::STATE_SHIPPED') %}
        <div class="item">
    -       <div class="right floated content">
    +       <div class="item ui segment" style="margin: 0 0 5px; padding: 5px;">
        ...
        {% if shipment.tracking is not empty %}
            <div class="ui segment">
                <span class="ui top attached icon label"><i class="plane icon"></i> {{ 'sylius.ui.tracking_code'|trans|upper }}</span>
                <p>{{ shipment.tracking }}</p>
            </div>
        {% endif %}
        
    +   {% if shipment.state == shipped %}
    +       {% set path = path('sylius_admin_shipment_resend_confirmation_email', {'id': shipment.id, '_csrf_token': csrf_token(shipment.id)}) %}
    +       <a href="{{ path }}" class="ui icon labeled tiny fluid button" style="margin: 7px 0 0;" {{ sylius_test_html_attribute('resend-shipment-confirmation-email') }}>
    +           <i class="send icon"></i> {{ 'sylius.ui.resend_the_shipment_confirmation_email'|trans }}
    +       </a>
    +   {% endif %}
        </div>
        ...
    ```

* Update file `templates/bundles/SyliusShopBundle/Account/Order/Show/_header.html.twig` by adding:

    ```diff
        ...
        <div class="item">
    -       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|locale }}
    +       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|locale_name }}
        </div>
        ...
     ```
  
* Update file `src/Resources/templates/bundles/SyliusAdminBundle/AdminUser/_form.html.twig` by adding:

    ```diff
        ...
                {{ form_row(form.plainPassword) }}
                {{ form_row(form.enabled) }}
            </div>
    -   </div>
    -   <div class="column">
            <div class="ui segment">
                <h4 class="ui dividing header">{{ 'sylius.ui.additional_information'|trans }}</h4>
        ...
                {% endif %}
            </div>
    +   </div>
    +   <div class="column">
            <div class="ui segment">
                <h4 class="ui dividing header">{{ 'sylius.ui.preferences'|trans }}</h4>
        ...
    +   <div class="ui segment">
    +       <h4 class="ui dividing header">{{ 'sylius.ui.channel'|trans }}</h4>
    +       {{ form_row(form.channel) }}
    +   </div>
        ...
     ```

## Migrations

```bash
cp vendor/sylius/refund-plugin/migrations/Version20200225100818.php src/Migrations
```

# UPGRADE FORM 0.15.0 to 0.16.0

## General update

```bash
composer require "sylius/plus:0.16.*"
```

* Upgrade SyliusRefundPlugin from `v0.10.1` to `v1.0.0-RC.2`

```bash
composer require "sylius/refund-plugin": "^1.0.0-RC.2"
```

## Update templates

* For fix broken `number_format_percent()` function. Override file `vendor/sylius/plus/src/Returns/Infrastructure/Resources/views/Admin/Order/ReturnRequest/Index/_returnRate.html.twig` and add:


    ```diff
        ...
         <div id="return-rate" class="value" style="padding-bottom: 10px;">
    -        {{ returnRate|number_format_percent() }}
    +        {{ returnRate|format_percent_number() }}
        </div>
        ...
    ```

* Update file `templates/bundles/SyliusAdminBundle/Order/Show/_header.html.twig` by adding:

    ```diff
        ...
        <div class="item">
    -       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|locale }}
    +       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|locale_name }}
        </div>
        ...
    ```

## Migrations

```bash
cp vendor/sylius/refund-plugin/migrations/Version20200306153205.php src/Migrations
cp vendor/sylius/refund-plugin/migrations/Version20200306145439.php src/Migrations
cp vendor/sylius/refund-plugin/migrations/Version20200310094633.php src/Migrations
cp vendor/sylius/refund-plugin/migrations/Version20200310185620.php src/Migrations
```

# UPGRADE FROM 0.14.* to 0.15.0

## General update

```bash
composer require "sylius/plus:0.15.*"
```

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20200125182414.php src/Migrations
cp vendor/sylius/plus/migrations/Version20200131082149.php src/Migrations
```

## Update templates

```bash
cp vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Shipment/Partial/_ship.html.twig templates/bundles/SyliusAdminBundle/Shipment/Partial/_ship.html.twig
```

# UPGRADE FROM 0.13.* to 0.14.0

## Important changes

Relation between `src/Entity/Channel` and `Sylius\Plus\BusinessUnits\Domain\Model\BusinessUnit` was changed from One-To-One to Many-To-One.

## General update

```bash
composer require "sylius/plus:0.14.*" --update-with-dependencies
```

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20191216134233.php src/Migrations
cp vendor/sylius/plus/migrations/Version20191230121402.php src/Migrations
cp vendor/sylius/plus/migrations/Version20200110100230.php src/Migrations
cp vendor/sylius/plus/migrations/Version20200113091731.php src/Migrations
```

## Update templates

```bash
cp templates/bundles/SyliusRefundPlugin/orderRefunds.html.twig templates/bundles/SyliusRefundPlugin
```

# UPGRADE FROM 0.12.* to 0.13.0

## General update

```bash
composer require "sylius/plus:0.13.*"
```

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20191216131849.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20191217075815.php src/Migrations/
```

# UPGRADE FROM 0.11.* to 0.12.0

## General update

```bash
composer require "sylius/plus:0.12.*" --no-update
composer config minimum-stability stable # to downgrade Sylius/RefundPlugin and Sylius/InvoicingPlugin to tagged versions
composer update
```

## SyliusPlusRbacPlugin-related changes

### Previous version installed with SyliusPlusRbacPlugin

If you had the `Sylius/SyliusPlusRbacPlugin` installed, you need to:

* Remove the plugin from `composer.json`:

    ```diff
    {
        "require": {
            ...
    -        "sylius/plus-rbac-plugin": "dev-master",
            ...
        },
    }
    ```

* Remove the plugin from `config/bundles.php` file:

    ```diff
    return [
        ...
    -    Sylius\Plus\RbacPlugin\SyliusPlusRbacPlugin::class => ['all' => true],
    ];
    ```

* Remove config import in `config/packages/_sylius.yaml` file:

    ```diff
    imports:
        ...
        
    -    - { resource: "@SyliusPlusRbacPlugin/Resources/config/config.yml" }
    
    -sylius_resource:
    -    authorization_checker: sylius_plus_rbac_plugin.authorization_checker.resource
    ```

* Remove routing import from the plugin in `config/routes/sylius_admin.yaml` file.

    ```diff
    -sylius_plus_rbac_plugin:
    -    resource: "@SyliusPlusRbacPlugin/Resources/config/routing.yml"
    -    prefix: /admin
    ```
    
* Remove implementing `RbacAdminUserInterfaceAlias` in `src/Entity/User/AdminUser.php` and modify the namespaces of traits:

    ```diff
    -use Sylius\Plus\RbacPlugin\Model\RoleableTrait;
    -use Sylius\Plus\RbacPlugin\Model\ToggleablePermissionCheckerTrait;
    +use Sylius\Plus\Rbac\Domain\Model\RoleableTrait;
    +use Sylius\Plus\Rbac\Domain\Model\ToggleablePermissionCheckerTrait;
    ```

* Update file import in `templates/bundles/SyliusUiBundle/Grid/_default.html.twig`:

    ```diff
    -{% import '@SyliusPlusRbacPlugin/Admin/Macro/table.html.twig' as table %}
    +{% import '@SyliusPlusPlugin/Resources/views/Admin/Macro/table.html.twig' as table %}
    ```

### Previous version installed without SyliusPlusRbacPlugin

If you didn't have the `Sylius/SyliusPlusRbacPlugin` installed, you need to modify your `src/Entity/User/AdminUser.php`
by adding required traits: 

```php
<?php

declare(strict_types=1);

namespace App\Entity\User;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Table;
use Sylius\Component\Core\Model\AdminUser as BaseAdminUser;
use Sylius\Plus\Entity\AdminUserInterface;
use Sylius\Plus\Entity\AdminUserTrait;
use Sylius\Plus\Rbac\Domain\Model\RoleableTrait;
use Sylius\Plus\Rbac\Domain\Model\ToggleablePermissionCheckerTrait;

/**
 * @Entity
 * @Table(name="sylius_admin_user")
 */
class AdminUser extends BaseAdminUser implements AdminUserInterface
{
    use AdminUserTrait;
    use ToggleablePermissionCheckerTrait;
    use RoleableTrait;

    public function __construct()
    {
        parent::__construct();

        $this->rolesResources = new ArrayCollection();
    }
}
```

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20191122134157.php src/Migrations/
```
