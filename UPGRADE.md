# UPGRADE FROM 1.0.0-BETA.4 to 1.0.0-BETA.5

1. Replace `sm_can` twigs function with `sylius_sm_can` in all templates.

# UPGRADE FROM 1.0.0-BETA.3 to 1.0.0-BETA.4

1. Support for Sylius 1.13 has been added, it is now the recommended Sylius version to use.

1. Support for Sylius 1.11 has been dropped, upgrade your application to [Sylius 1.12](https://github.com/Sylius/Sylius/blob/1.12/UPGRADE-1.12.md).
   or to [Sylius 1.13](https://github.com/Sylius/Sylius/blob/1.13/UPGRADE-1.13.md).

1. Support for PHP 8.0 has been dropped.

1. The following constructor signatures have been changed:

   `Sylius\Plus\Inventory\Application\DataPersister\ProductVariantDataPersister`
    ```diff
        public function __construct(
        -   ContextAwareDataPersisterInterface $decoratedDataPersister,
            FactoryInterface $inventorySourceStockFactory,
            RepositoryInterface $inventorySourceRepository,
        )
    ```

1. The following classes have transitioned from using `SM\Factory\FactoryInterface` to `Sylius\Abstraction\StateMachine\StateMachineInterface`. However, to ensure backward compatibility, their constructors still accept `SM\Factory\FactoryInterface`:

    - `Sylius\Plus\PartialShipping\Application\CommandHandler\SplitAndSendShipmentHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\AcceptReturnRequestHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\CancelReturnRequestHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\MarkReturnRequestAsItemsReturnedToInventoryHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\MarkReturnRequestAsPackageReceivedHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\MarkReturnRequestAsRepairedItemsSentHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\MarkReturnRequestUnitsAsReceivedHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\RejectReturnRequestHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\ResolveReturnRequestHandler`
    - `Sylius\Plus\Returns\Application\CommandHandler\ReturnUnitsToInventoryHandler`
    - `Sylius\Plus\Returns\Application\Creator\ReplacementOrderCreator`
    - `Sylius\Plus\Returns\Application\StateResolver\ReturnRequestItemsReturnedToInventoryStateResolver`
    - `Sylius\Plus\Returns\Application\StateResolver\ReturnRequestPackageReceivedStateResolver`
    - `Sylius\Plus\Returns\Infrastructure\Ui\Admin\SelectReturnRequestUnitsAsReceivedAction`
    - `Sylius\Plus\Returns\Infrastructure\Ui\Admin\SelectUnitsToReturnToInventoryAction`
    - `Sylius\Plus\Returns\Infrastructure\Ui\Shop\CancelReturnRequestAction`

# UPGRADE FROM 1.0.0-BETA.2 to 1.0.0-BETA.3

1. The constructor of `Sylius\Plus\Inventory\Infrastructure\Ui\SplitShipmentAction` has been changed:
    
    ```diff
        public function __construct(
        +   private SplitAndSendShipmentCommandCreatorInterface $commandCreator,
            private MessageBusInterface $commandBus,
            private ShipmentRepositoryInterface $shipmentRepository,
            private RequestStack $requestStack,
            private UrlGeneratorInterface $router,
            private FormFactoryInterface $formFactory,
            private Environment $twig,
        +   private ValidatorInterface $validator,
        +   private array $validationGroups,
        ) {
        }
    ```

2. The constructor of `Sylius\Plus\PartialShipping\Application\CommandHandler\SplitAndSendShipmentHandler` has been changed:
    
    ```diff
        public function __construct(
        -   private ShipmentFactoryInterface $shipmentFactory,
        -   private AdjustmentDuplicatorInterface $adjustmentDuplicator,
            private ShipmentRepositoryInterface $shipmentRepository,
        -   private OrderItemUnitRepositoryInterface $orderItemUnitRepository,
        -   private RepositoryInterface $inventorySourceRepository,
            private ObjectManager $shipmentManager,
        -   private VariantsQuantityMapFactoryInterface $variantsQuantityMapFactory,
        -   private ChangeInventorySourceOperatorInterface $changeInventorySourceOperator,
            private FactoryInterface $stateMachineFactory,
        +   private ShipmentSplitterInterface $shipmentSplitter,
        ) {
        }
    ```

3. The constructor of `\Sylius\Plus\Returns\Infrastructure\Ui\Admin\CreateReplacementOrderAction` has been changed:

    ```diff
        public function __construct(
            private MessageBusInterface $commandBus,
        -   private RepositoryInterface $orderRepository,
            private RequestStack $requestStack,
            private UrlGeneratorInterface $router,
            private FormFactoryInterface $formFactory,
            private Environment $twig,
            private RepositoryInterface $returnRequestRepository,
        -   private VariantsQuantityMapFactoryInterface $variantsQuantityMapFactory,
        +   private ReplacementOrderFactoryInterface $replacementOrderFactory,
        ) {
        }
    ```

4. The constructor of `Sylius\Plus\Returns\Infrastructure\Ui\Admin\ChangeReturnRequestResolutionAction` has been changed:

    ```diff
        public function __construct(
        +   private FormFactoryInterface $formFactory,
            private MessageBusInterface $commandBus,
        +   private ChangeReturnRequestResolutionCommandCreatorInterface $commandCreator,
            private UrlGeneratorInterface $router,
            private RequestStack $requestStack,
        +   private ValidatorInterface $validator,
        +   private array $validationGroups,
        -   private CsrfCheckerInterface $csrfChecker,
        ) {
        }
    ```

5. The `Sylius\Plus\Inventory\Application\Applier\InventorySourceStockApplier` and
   `Sylius\Plus\Inventory\Application\Applier\InventorySourceStockApplierInterface` have been removed.

6. The `Sylius\Plus\Inventory\Infrastructure\Ui\ApplyInventorySourceStockForProductVariantAction` has been refactored into
   `Sylius\Plus\Inventory\Infrastructure\Ui\ChangeInventorySourceStockOnHandAction` and its constructor now looks like this:

    ```diff
         public function __construct(
              private RepositoryInterface $inventorySourceRepository,
              private EntityManagerInterface $inventorySourceManager,
              private FormFactoryInterface $formFactory,
              private UrlGeneratorInterface $router,
              private RequestStack $requestStack,
              private Environment $twig,
         ) {
         }
    ```

7. The route for updating particular inventory stock on hand level has been updated:

    ```diff
    -   sylius_admin_inventory_source_stock_applier:
    -      path: /inventory-sources/{id}/manage/update-stock
    -      methods: [POST]
    -      defaults:
    -          _controller: Sylius\Plus\Inventory\Infrastructure\Ui\ApplyInventorySourceStockForProductVariantAction
    +   sylius_admin_inventory_source_stock_change_on_hand:
    +      path: /inventory-sources/{id}/manage/{stockId}/update-stock
    +      methods: [GET, POST]
    +      defaults:
    +          _controller: Sylius\Plus\Inventory\Infrastructure\Ui\ChangeInventorySourceStockOnHandAction
    ```

8. Form data transformers:
    - `Sylius\Plus\Loyalty\Infrastructure\Form\DataTransformer\ChangeActionToChannelsBasedItemsTotalToPointsRatioConfigurationTransformer`
    - `Sylius\Plus\Loyalty\Infrastructure\Form\DataTransformer\ChangeActionToChannelsBasedPointsPerProductRatioConfigurationTransformer`
      have been removed.

   In their place a generic `Sylius\Plus\Loyalty\Infrastructure\Form\DataTransformer\ChangeActionToChannelsBasedRatioConfigurationTransformer`
   has been added to ease customization.

9. The following templates have been removed in favour of using Sylius template events:
    - src/Resources/templates/bundles/SyliusRefundPlugin/_header.html.twig
    - src/Resources/templates/bundles/SyliusRefundPlugin/_unitInput.html.twig
    - src/Resources/templates/bundles/SyliusRefundPlugin/orderRefunds.html.twig

   You need to remove it from your side and if you already override some of these templates
   you can refactor it to use events or keep overriding it this way.

10. The `src/Resources/views/Shipment/shipmentSplit.html.twig` template has been adjusted to use Sylius Template events.
   Content of this template has been divided and moved to both `src/Resources/views/Shipment/Header` and `src/Resources/views/Shipment/Form` catalogs.

   ##### Moreover some of the existing templates have been moved:

    * `src/Resources/views/Shipment/_breadcrumb.html.twig` has been moved to `src/Resources/views/Shipment/Header/_breadcrumb.html.twig`
    * `src/Resources/views/Shipment/_headerTitle.html.twig` has been moved to `src/Resources/views/Shipment/Header/_headerTitle.html.twig`
    * `src/Resources/views/Shipment/_header.html.twig` has been moved to `src/Resources/views/Shipment/Header/_header.html.twig`

# UPGRADE FROM 1.0.0-BETA.1 to 1.0.0-BETA.2

1. The `Sylius\Plus\Entity\ReturnRequest` and `Sylius\Plus\Entity\CreditMemoAwareTrait` have been removed and the methods
   `public function creditMemos(): Collection` and `public function addCreditMemo(CreditMemoInterface $creditMemo): void` 
   have been moved back to the `Sylius\Plus\Returns\Domain\Model\ReturnRequest`.

2. The `sylius.validator.stock_on_hand` service has been renamed into `sylius.validator.stock_on_hand_cannot_be_lower_than_on_hold`
   and it points to the `Sylius\Plus\Inventory\Infrastructure\Validator\StockOnHandCannotBeLowerThanOnHoldValidator` class.

3. The `Sylius\Plus\Inventory\Infrastructure\Validator\StockOnHandOnHold` validation constraint has been renamed
   into `Sylius\Plus\Inventory\Infrastructure\Validator\StockOnHandCannotBeLowerThanOnHold`
   and it's now validated by `sylius_stock_on_hand_cannot_bo_lower_than_on_hold_validator`.

4. The `Sylius\Plus\BusinessUnits\Domain\Model\BusinessUnitAddress` has been transformed into a resource for better extendability,
   it now resides in its own table `sylius_business_unit_address` instead of being embedded within the `sylius_business_unit`.

# UPGRADE FROM 1.0.0-ALPHA.9 to 1.0.0-BETA.1

1. The constructor of `Sylius\Plus\Rbac\Infrastructure\Templating\Helper\AclHelper` class has been changed:

    ```diff
        public function __construct(
            protected AdminPermissionResolverInterface $adminPermissionResolver,
    +       protected AdminUserContextInterface $adminUserContext,
        ) {
        }
    ```

# UPGRADE FROM 1.0.0-ALPHA.8 to 1.0.0-ALPHA.9

1. The constructor of `Sylius\Plus\PartialShipping\Application\CommandHandler\SplitAndSendShipmentHandler` class has been changed:

    ```diff
        public function __construct(
            private ShipmentFactoryInterface $shipmentFactory,
    +       private AdjustmentDuplicatorInterface $adjustmentDuplicator,
            private ShipmentRepositoryInterface $shipmentRepository,
            private OrderItemUnitRepositoryInterface $orderItemUnitRepository,
            private RepositoryInterface $inventorySourceRepository,
            private ObjectManager $shipmentManager,
            private VariantsQuantityMapFactoryInterface $variantsQuantityMapFactory,
            private ChangeInventorySourceOperatorInterface $changeInventorySourceOperator,
            private FactoryInterface $stateMachineFactory,
        ) {
        }
    ```

1. The constructor of `Sylius\Plus\OAuth\UserProvider` class has been changed:

    ```diff
        public function __construct(
            string $supportedUserClass,
    -       FactoryInterface $userFactory,
            UserRepositoryInterface $userRepository,
    -       FactoryInterface $oauthFactory,
    -       RepositoryInterface $oauthRepository,
    -       ObjectManager $userManager,
            CanonicalizerInterface $canonicalizer,
    -       CustomerProviderInterface $customerProvider,
    +       private FactoryInterface $userFactory,
    +       private FactoryInterface $oauthFactory,
    +       private RepositoryInterface $oauthRepository,
    +       private ObjectManager $userManager,
    +       private CustomerResolverInterface $customerResolver,
        ) {
            parent::__construct($supportedUserClass, $userRepository, $canonicalizer);

    -       $this->oauthFactory = $oauthFactory;
    -       $this->oauthRepository = $oauthRepository;
    -       $this->userFactory = $userFactory;
    -       $this->userManager = $userManager;
    -       $this->customerProvider = $customerProvider;
        }
    ```
   
1. The `Symfony\Component\HttpFoundation\Session\Session` argument has been replaced with `Symfony\Component\HttpFoundation\RequestStack` in the constructor of following actions:
    * `Sylius\Plus\Inventory\Infrastructure\Ui\ApplyInventorySourceStockForProductVariantAction`
    * `Sylius\Plus\Inventory\Infrastructure\Ui\SplitShipmentAction`
    * `Sylius\Plus\Loyalty\Infrastructure\Ui\Shop\BuyLoyaltyPurchaseAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\AcceptReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\ChangeReturnRequestResolutionAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\CreateReplacementOrderAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\MarkReturnRequestAsRepairedItemsSentAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\MarkReturnRequestUnitsAsReceivedAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\RefundUnitsAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\RejectReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\ResolveReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\ReturnUnitsToInventoryAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\SelectReturnRequestUnitsAsReceivedAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\SelectUnitsToReturnToInventoryAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\SplitReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Shop\CancelReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Shop\RequestReturnAction`

# UPGRADE FROM 1.0.0-ALPHA.7 to 1.0.0-ALPHA.8

1. Support for Sylius v1.10 has been dropped, you need to upgrade Sylius to v1.11.0. 
   Please follow [Sylius's upgrade instructions](https://github.com/Sylius/Sylius/blob/master/UPGRADE-1.11.md).

1. Doctrine string functions were removed from entity manager, due to collisions with Postgres.

1. Classes and interfaces have been moved to new namespaces:
    * `Sylius\Plus\BusinessUnits\Application\Listener\InvoiceShopBillingDataFactory` to `Sylius\Plus\Factory\InvoiceShopBillingDataFactory`
    * `Sylius\Plus\BusinessUnits\Application\Listener\InvoiceShopBillingDataFactoryInterface` to `Sylius\Plus\Factory\InvoiceShopBillingDataFactoryInterface`
    * `Sylius\Plus\ChannelAdmin\Application\Checker\CreditMemoResourceChannelChecker` to `Sylius\Plus\Checker\CreditMemoResourceChannelChecker`
    * `Sylius\Plus\ChannelAdmin\Application\Checker\CustomerResourceChannelChecker` to `Sylius\Plus\CustomerPools\Application\Checker\CustomerResourceChannelChecker`
    * `Sylius\Plus\ChannelAdmin\Application\Checker\InvoiceResourceChannelChecker` to `Sylius\Plus\Checker\InvoiceResourceChannelChecker`
    * `Sylius\Plus\ChannelAdmin\Application\Checker\ResourceChannelCheckerInterface` to `Sylius\Plus\SharedKernel\ResourceChannelCheckerInterface`
    * `Sylius\Plus\ChannelAdmin\Application\Checker\ReturnRequestResourceChannelChecker` to `Sylius\Plus\Returns\Application\Checker\ReturnRequestResourceChannelChecker`
    * `Sylius\Plus\ChannelAdmin\Application\Exception\ResourceNotSupportedException` to `Sylius\Plus\SharedKernel\ResourceNotSupportedException`
    * `Sylius\Plus\Inventory\Application\Operator\ReturnInventoryOperator` to `Sylius\Plus\Operator\ReturnInventoryOperator`
    * `Sylius\Plus\Inventory\Application\Operator\ReturnInventoryOperatorInterface` to `Sylius\Plus\Returns\Application\Operator\ReturnInventoryOperatorInterface`
    * `Sylius\Plus\Returns\Infrastructure\Listener\CreditMemoWithReturnRequestEventListener` to `Sylius\Plus\Listener\CreditMemoWithReturnRequestEventListener`
    * `Sylius\Plus\Reviews\Infrastructure\Validator\ApiUniqueReviewerEmailValidator` to `Sylius\Plus\CustomerPools\Infrastructure\Validator\ApiUniqueReviewerEmailValidator`
    * `Sylius\Plus\Reviews\Infrastructure\Validator\UniqueReviewerEmailValidator` to `Sylius\Plus\CustomerPools\Infrastructure\Validator\UniqueReviewerEmailValidator`

1. The `Sylius\Plus\Returns\Domain\Model\ReturnRequest` class has been changed:
    * `public function creditMemos(): Collection` and `public function addCreditMemo(CreditMemoInterface $creditMemo): void` methods 
      have been moved to `Sylius\Plus\Entity\CreditMemoAwareTrait`
    * The `Sylius\Plus\Entity\ReturnRequest` is extending `Sylius\Plus\Returns\Domain\Model\ReturnRequest` and using `CreditMemosAwareTrait` 
      to achieve the previous behaviour and is now in use. If you want to change it - you have to replace this new entity.

1. The `Sylius\Plus\Returns\Application\Creator\ReplacementOrderCreatorInterface` interface has been changed:

    ```diff
    +   public function createFromReturnRequest(ReturnRequestInterface $returnRequest, array $variantsCodeWithQuantity): OrderInterface;
    -   public function createFromReturnRequest(ReturnRequestInterface $returnRequest, VariantsQuantityMapInterface $variantsQuantityMap): OrderInterface;
    ```

1. The constructor of `Sylius\Plus\Returns\Application\Creator\ReplacementOrderCreator` class has been changed:

    ```php
        public function __construct(
    -       private VariantsQuantityMapFactoryInterface $variantsQuantityMapFactory,
            private FactoryInterface $orderFactory,
            private FactoryInterface $orderItemFactory,
            private FactoryInterface $shipmentFactory,
    -       private VariantQuantityMapAvailabilityCheckerInterface $availabilityChecker,
    +       private OrderItemsAvailabilityCheckerInterface $orderItemsAvailabilityChecker,
            private OrderItemQuantityModifierInterface $orderItemQuantityModifier,
    +       private ProductVariantRepository $productVariantRepository,
            private StateMachineFactoryInterface $stateMachineFactory
        ) {
        }
    ```

1. The fourth constructor parameter of `Sylius\Plus\Returns\Application\CommandHandler\CreateReplacementOrderHandler` has been removed.

1. The `Sylius\Plus\Loyalty\Infrastructure\Form\Type\ProductPerChannelChoiceType` has been replaced with `Sylius\Plus\Loyalty\Infrastructure\Form\Type\ProductPerChannelAutocompleteChoiceType`.

1. The `Sylius\Plus\Doctrine\ORM\FindEnabledProductsByChannelQueryInterface` and its implementation `Sylius\Plus\Doctrine\ORM\FindEnabledProductsByChannelQuery` have been removed.
   Therefore, the constructor of `Sylius\Plus\Inventory\Infrastructure\Fixture\InventorySourceStockFixture` takes
   `Sylius\Component\Core\Repository\ProductRepositoryInterface` in place of `Sylius\Plus\Doctrine\ORM\FindEnabledProductsByChannelQueryInterface`.

1. The entire GMV report feature has been removed. All used services from this feature should be replaced by a custom implementation.

1. `Sylius\Plus\Returns\Infrastructure\Checker\CsrfCheckerInterface` has been added as a last argument to constructor for controllers:
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\AcceptReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\ChangeReturnRequestResolutionAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\CreateReplacementOrderAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\MarkReturnRequestUnitsAsReceivedAction`
    * `Sylius\RefundPlugin\Action\Admin\RefundUnitsAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\RejectReturnRequestAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\MarkReturnRequestAsRepairedItemsSentAction`
    * `Sylius\Plus\Returns\Infrastructure\Ui\Admin\ResolveReturnRequestAction`

1. The constructor of `Sylius\Plus\Loyalty\Application\Processor\LoyaltyPointsProcessorInterface` class has been changed:

    ```php
        public function __construct(
            private AdjustmentFactoryInterface $adjustmentFactory,
    -       private RepositoryInterface $loyaltyRuleRepository,
    +       private LoyaltyRuleQueryInterface $loyaltyRuleQuery,
            private ActionBasedLoyaltyPointsCalculatorInterface $calculator
        ) {
        }
    ```

# UPGRADE FROM 1.0.0-ALPHA.6 to 1.0.0-ALPHA.7

## General update

1. It was introduced the way to disable PDF generation. Please note that using the configuration below
   will not allow you to generate the Gross Merchandise Value Report and Return Request PDFs.

```yaml
sylius_plus:
    pdf_generator:
        enabled: false
```        
 
1. The admin template overriding method has been changed to use Sylius template events. You need to remove it by your side 
   and if you already override some of these templates you can refactor it to use events or keep overriding it this way.
    * `src/Resources/templates/bundles/SyliusAdminBundle/*` has been removed and replaced using events. Deleted templates:
        * `src/Resources/templates/bundles/SyliusAdminBundle/AdminUser/_form.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Channel/_form.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Customer/_form.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Dashboard/_channelSwitch.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Layout/_logo.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Order/Show/_header.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Order/Show/_payment.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Order/Show/_shipment.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Order/Show/_shipments.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Product/Show/_details.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Product/Show/_simpleProduct.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Product/Show/_variantContent.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Product/Show/_variantItem.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Product/Show/_variants.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Product/Tab/_inventory.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/ProductVariant/Tab/_inventory.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Shipment/Grid/Action/shipWithTrackingCode.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Shipment/Partial/_ship.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Security/_content.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Shipment/Show/_breadcrumb.html.twig`
        * `src/Resources/templates/bundles/SyliusAdminBundle/Shipment/_show.html.twig`
        
1. Classes have been moved to new namespaces:
   * `Sylius\Plus\Calculator\ShippingCalculator\DelegatingCalculator` to `Sylius\Plus\Returns\Application\Calculator\ShippingCalculator\DelegatingCalculator`
   * `Sylius\Plus\Controller\OrderItemController` to `Sylius\Plus\Inventory\Application\Controller\OrderItemController`
   * `Sylius\Plus\EventListener\ChannelCreateListener` to `Sylius\Plus\Inventory\Application\EventListener\ChannelCreateListener`
   * `Sylius\Plus\EventListener\ReviewCreateListener` to `Sylius\Plus\CustomerPools\Application\EventListener\ReviewCreateListener`

1. `Sylius\Plus\Returns\Application\CommandHandler\RequestReturnHandler` now uses factory to create ReturnRequest instead of 'new' statement.
    And this service has extra argument on constructor (`Sylius\Plus\Returns\Application\Factory\ReturnRequestFactoryInterface`).

1. `Sylius\Plus\EventListener\AuthenticationSuccessListener` service has been removed. Use `sylius.listener.api_authentication_success_listener` 
    from Sylius instead.

# UPGRADE FROM 1.0.0-ALPHA.5 to 1.0.0-ALPHA.6

## General update

1. The way of customizing resource definition has been changed.

   Before:

    ```yaml
        sylius_resource:
            resources:
                sylius_plus.sample_resource:
                    ...
    ```  

   After:

    ```yaml
        sylius_plus:
            resources:
                sample_resource:
                    ...
    ```

1. The repositories that overwrite the repositories from Sylius have been refactored:
    * The `Sylius\Plus\Doctrine\ORM\CountCustomersQuery` and `Sylius\Plus\ChannelAdmin\Infrastructure\Doctrine\ORM\CustomerListQueryBuilder`
      have been extracted from the `Sylius\Plus\Doctrine\ORM\CustomerRepository`.
    * The `Sylius\Plus\Doctrine\ORM\CreditMemoRepository` has been removed and replaced by `Sylius\Plus\ChannelAdmin\Infrastructure\Doctrine\ORM\CreditMemoListQueryBuilder`.
    * The `Sylius\Plus\Doctrine\ORM\InvoiceRepository` has been removed and replaced by `Sylius\Plus\ChannelAdmin\Infrastructure\Doctrine\ORM\InvoiceListQueryBuilder`.
    * The `Sylius\Plus\Doctrine\ORM\OrderItemUnitRepository` has been removed and replaced by `Sylius\Plus\Returns\Infrastructure\Doctrine\ORM\CountOrderItemUnitsQuery`.
    * The `Sylius\Plus\Doctrine\ORM\ProductRepository` has been removed and replaced by `Sylius\Plus\Doctrine\ORM\FindEnabledProductsByChannelQuery`
      and `Sylius\Plus\Inventory\Infrastructure\Doctrine\ORM\FindAllDescendantProductsByTaxonQuery`.
    * The `Sylius\Plus\Doctrine\ORM\ProductVariantRepository` has been removed and replaced by `Sylius\Plus\Doctrine\ORM\FindProductVariantsByNameInChannelQuery`.
    * The `Sylius\Plus\Doctrine\ORM\ShopUserRepository` has been removed and replaced by `Sylius\Plus\CustomerPools\Infrastructure\Doctrine\ORM\FindShopUserByUsernameAndCustomerPoolQuery`.
    * The `Sylius\Plus\Doctrine\ORM\ShopUserRepository::findOneByEmailAndCustomerPool` method has been removed,
      use `Sylius\Plus\CustomerPools\Infrastructure\Doctrine\ORM\FindShopUserByUsernameAndCustomerPoolQuery` instead.

1. The entity traits and interfaces have been refactored to split them between different contexts:
    * The `Sylius\Plus\Entity\AdminUserTrait` has been removed, use `Sylius\Plus\ChannelAdmin\Domain\Model\AdminChannelAwareTrait`,
      `Sylius\Plus\Rbac\Domain\Model\ToggleablePermissionCheckerTrait`, `Sylius\Plus\Rbac\Domain\Model\RoleableTrait` and `Sylius\Plus\Entity\LastLoginIpAwareTrait` instead.
    * The `Sylius\Plus\Entity\AdminUserInterface` has been removed, use `Sylius\Plus\Rbac\Domain\Model\AdminUserInterface`,
      `Sylius\Component\Channel\Model\ChannelAwareInterface` and `Sylius\Plus\Entity\LastLoginIpAwareInterface` instead.
    * The `Sylius\Plus\Entity\ProductVariantTrait` has been removed, use `Sylius\Plus\Inventory\Domain\Model\InventorySourceStocksAwareTrait` instead.
    * The `Sylius\Plus\Entity\ProductVariantInterface` has been removed, use `Sylius\Plus\Inventory\Domain\Model\ProductVariantInterface` instead.
    * The `Sylius\Plus\Entity\AdjustmentInterface` has been removed, use `Sylius\Component\Core\Model\AdjustmentInterface`,
      `Sylius\Plus\Loyalty\Domain\Model\AdjustmentType` and `Sylius\Plus\Returns\Domain\Model\AdjustmentType` instead.
    * The `Sylius\Plus\Entity\CustomerTrait` has been removed, use `Sylius\Plus\CustomerPools\Domain\Model\CustomerPoolAwareTrait`
      and `Sylius\Plus\Loyalty\Domain\Model\LoyaltyAwareTrait` instead.
    * The `Sylius\Plus\Entity\CustomerInterface` has been removed, use `Sylius\Plus\CustomerPools\Domain\Model\CustomerInterface`
      and `Sylius\Plus\Loyalty\Domain\Model\CustomerInterface` instead.
    * The `Sylius\Plus\Entity\ShipmentTrait` has been removed, use `Sylius\Plus\Inventory\Domain\Model\InventorySourceAwareTrait` instead.
    * The `Sylius\Plus\Entity\ShipmentInterface` has been removed, use `Sylius\Plus\Inventory\Domain\Model\ShipmentInterface` instead.
    * The `Sylius\Plus\Entity\ChannelTrait` has been removed, use `Sylius\Plus\CustomerPools\Domain\Model\CustomerPoolAwareTrait`,
      `Sylius\Plus\Returns\Domain\Model\ReturnRequestsAllowedAwareTrait` and `Sylius\Plus\BusinessUnits\Domain\Model\BusinessUnitAwareTrait` instead.
    * The `Sylius\Plus\Entity\ChannelInterface` has been removed, use `Sylius\Plus\BusinessUnits\Domain\Model\ChannelInterface`,
      `Sylius\Plus\Returns\Domain\Model\ChannelInterface` and `Sylius\Plus\CustomerPools\Domain\Model\ChannelInterface` instead.
    * The `Sylius\Plus\Entity\OrderTrait` has been removed, use `Sylius\Plus\Returns\Domain\Model\ReturnRequestAwareTrait`
      and `Sylius\Plus\Loyalty\Application\Provider\OrdersLoyaltyPointsProvider` instead.
    * The `Sylius\Plus\Entity\OrderInterface` has been removed, use `Sylius\Plus\Returns\Domain\Model\OrderInterface` instead.

1. The `Sylius\Plus\Returns\Domain\Model\ReturnRequestUnit` has been made a resource.
   Based on this change, `getId()` method was added and `id()` method is now marked as `@deprecated` as it will be removed in v1.0.0.

1. The `Sylius\Plus\Returns\Application\Mapper\ReturnRequestUnitMapper` has been modified to use new `Sylius\Plus\Returns\Application\Factory\ReturnRequestUnitFactory`.
   Due to this change, a new argument has been added to the constructor:

    ```diff
        public function __construct(
            Private RepositoryInterface $orderItemUnitRepository,
    +       Private ReturnRequestUnitFactoryInterface $returnRequestUnitFactory
        ) {
        }
    ```

1. The `Sylius\Plus\Returns\Application\CommandHandler\RequestReturnHandler` logic and constructor has been changed due to refactor.

    ```diff
        public function __construct(
            private ObjectManager $returnRequestManager,
    -       private ReturnRequestConfirmationSender $returnRequestConfirmationSender, 
            ...
            private ImageUploaderInterface $uploader,
    +       private MessageBusInterface $commandBus
        ) {
            ...
        }
   ```

1. The `Sylius\Plus\ChannelAdmin\Application\Checker\ResourceChannelChecker` has been refactored to take in constructor
   list of checkers tagged by `sylius_plus.channel_admin.resource_channel_checker` and use them to check the given resource
   instead of doing it in its content.

1. The `Sylius\Plus\ChannelAdmin\Application\Checker\ResourceChannelEnabilibityChecker` has been refactored to take
   in constructor `sylius_plus.channel_admin.restricted_resources` parameter instead of using hardcoded list of resources in its content.

1. The `OrdersLoyaltyPointsProviderInterface $loyaltyPointsProvider` has been added as the 2nd argument
   in the constructor of `Sylius\Plus\Loyalty\Application\Assigner\LoyaltyPointsAssigner`

## Templates

1. The shop template overriding method has been changed to use Sylius template events:
    * The `src/Resources/templates/bundles/SyliusShopBundle/Account/Order/Show/_header.html.twig` along with `SyliusShopBundle/Account/Order/Show/` directories have been removed and replaced by using `header` block in `sylius.shop.account.order.show.subcontent` template event. Remove that file and directories from your `templates/bundles` directory or adjust it to your customizations.
    * The `sylius_plus.block_event_listener.shop_account_order_show.return_requests` Sonata block has been removed and replaced by `return_requests` block in `sylius.shop.account.order.show.subcontent` template event.
    * The `sylius_plus.block.block_event_listener.shop.javascripts` Sonata block has been removed and replaced by `javascripts` block in `sylius.shop.layout.javascripts` template event.

1. The `src/Resources/views/Loyalty/Shop/_loyaltyPurchases.html.twig` template has been renamed to `src/Resources/views/Loyalty/Shop/_loyalty.html.twig`.
   Due to this change, the `sylius_plus_shop_loyalty_purchase_index` route configuration has been modified:

    ```diff
         sylius_plus_shop_loyalty_purchase_index:
             path: /account/loyalty
             methods: [GET]
             defaults:
                 _controller: sylius_plus.controller.loyalty_purchase:indexAction
                 _sylius:
    -                template: '@SyliusPlusPlugin/Loyalty/Shop/_loyaltyPurchases.html.twig'
    +                template: '@SyliusPlusPlugin/Loyalty/Shop/_loyalty.html.twig'
                     grid: sylius_plus_shop_loyalty_purchase
    ```

# UPGRADE FROM 1.0.0-ALPHA.4 to 1.0.0-ALPHA.5

1. Since 1.0.0-ALPHA.5, the recommended Sylius version to use with Plus is `1.11.*`. If you would like to upgrade Sylius to v1.11.0, 
please follow [Sylius's upgrade instructions](https://github.com/Sylius/Sylius/blob/master/UPGRADE-1.11.md) and [Sylius API's upgrade instructions](https://github.com/Sylius/Sylius/blob/master/UPGRADE-API-1.11.md).

1. If you still use Sylius `1.10.*`, you need to override some resources api configurations. 
You can find them in `vendor/sylius/plus/etc/sylius-1.10/Resources/config/api_resources/`.

    If you did not change anything in these configurations, use following command to override required files:

    ```bash
    cp -R vendor/sylius/plus/etc/sylius-1.10/Resources/config/api_resources/* config/api_platform/
    rm vendor/sylius/plus/src/Resources/config/api_resources/Customer.xml
    rm vendor/sylius/plus/src/Resources/config/api_resources/Order.xml
    ```

1. Following services ids have been changed:
    * `Sylius\Plus\CustomerPools\Application\CommandHandler\SendAccountRegistrationEmailHandler` => `Sylius\Plus\CustomerPools\Application\CommandHandler\Account\SendAccountRegistrationEmailHandler`
    * `Sylius\Plus\CustomerPools\Application\CommandHandler\SendAccountVerificationEmailHandler` => `Sylius\Plus\CustomerPools\Application\CommandHandler\Account\SendAccountVerificationEmailHandler` 

# UPGRADE FROM 1.0.0-ALPHA.3 to 1.0.0-ALPHA.4

1. Controllers from `Inventory` and `Returns` namespaces have been refactored to explicitly get request parameters 
from the appropriate bag.

# UPGRADE FROM 1.0.0-ALPHA.2 to 1.0.0-ALPHA.3

1. The return types of `Sylius\Plus\Loyalty\Domain\Model\LoyaltyPointsTransactionInterface` methods have been changed:

    ```diff
    +   public function getType(): ?string;
    -   public function getType(): string;
    +   public function getPointsValue(): ?string;
    -   public function getPointsValue(): string;
   ```
 
1. The return types of `Sylius\Plus\Returns\Application\Command\RequestReturn` methods have been changed:

    ```diff
    +   public function orderNumber(): ?string;
    -   public function orderNumber(): string;
    +   public function currencyCode(): ?string;
    -   public function currencyCode(): string;
   ```

# UPGRADE FROM 1.0.0-ALPHA.1 to 1.0.0-ALPHA.2

## Potential BC-breaks

* Callbacks declared on state machine in `Returns/Infrastructure/Resources/config/state_machine/` and files
  `Inventory/Infrastructure/Resources/config/state_machine.yaml`,
  `PartialShipping/Infrastructure/Resources/config/state_machine.yml`,
  `Resources/config/config.yaml`,
  have now priorities declared, if you modified them in past this could lead to potential BC break.
  Please adjust your priorities if needed. Note that those priorities are executed in ascending order.

## General update

```bash
composer require "sylius/plus:1.0.0-ALPHA.2"
```

* `ShipmentInventorySourceAssignerInterface $shipmentInventorySourceAssigner` has been added as the 4th argument 
  in the constructor of `Sylius\Plus\Inventory\Application\Operator\InventoryOperator`

* Endpoints with changed code to IRI:

POST on `/api/v2/shop/account/loyalty-points-transactions`:

````
{
    - "loyaltyPurchaseCode": "string",
    + "loyaltyPurchase": "string",
}
````

* The `Sylius\Plus\Converter\InvoiceShopBillingDataConverter` has been removed in favor of decorating the factory
   `Sylius\Plus\BusinessUnits\Application\Factory`.

# UPGRADE FROM 0.40.0 to 1.0.0-ALPHA.1

## General update
```bash
composer require "sylius/plus:1.0.0-ALPHA.1"
```

* Since 1.0.0-ALPHA.1, the recommended Sylius version to use with SyliusPlus is `1.10.*`. If you still use Sylius `1.9.*`, you need to
  override some resources api configurations. You can find them in `vendor/sylius/plus/etc/sylius-1.9/Resources/config/api_resources/`.

## Changes for 1.9.*

  Changes summary:
    - `Customer.xml` - added slash after `customers` in line 24
    - `LoyaltyPurchase.xml` - all `loyalty-purchases/{code}` changed to `loyalty-purchases/{id}`
    - `Order.xml` - all `orders/{tokenValue}` changed to `orders/{id}`
    - `ProductVariant.xml` - all `product-variants/{code}` changed to `product-variants/{id}`
    - `Shipment.xml` - `admin_ship` item operation changed to use service method rather than custom command (line 105)
    ```xml
      <!-- Before -->
      <attribute name="messenger">input</attribute>
      <attribute name="input">Sylius\Bundle\ApiBundle\Command\Checkout\ShipShipment</attribute>
      <!-- After -->
      <attribute name="controller">sylius.api.shipment_state_machine_transition_applicator::ship</attribute>
    ```
    - `OrderItem.xml` - `<property name="variant" readableLink="false" writableLink="false"/>` property added to display variant
      as IRI

  If you did not change anything in these configurations, use following command to override required files:

  ```bash
  cp -R vendor/sylius/plus/etc/sylius-1.9/Resources/config/api_resources/* config/api_platform/
  rm vendor/sylius/plus/src/Resources/config/api_resources/Shipment.xml
  ```

* Remove deprecated api routing:

  .. code-block:: yaml

        // config/routes/sylius_admin_api.yaml:
        #...

        sylius_plus_admin_api:
            resource: "@SyliusPlusPlugin/Resources/config/api_routing.yaml"
            prefix: /api/v1

# UPGRADE FROM 0.39.0 to 0.40.0

## General update
```bash
composer require "sylius/plus:0.40.*"
```

## Buses
* We decided to unify the naming of all message buses across Sylius products.
  Command buses `sylius_plus.inventory.command_bus`, `sylius_plus.returns.command_bus` and `sylius_plus.loyalty.command_bus` have been replaced with `sylius.command_bus`.

## InventorySourceStockUpdater
* Methods `update` and `increment` of `Sylius\Plus\Inventory\Application\Updater\InventorySourceStockUpdaterInterface` have changed return types from `void` to `InventorySourceStockInterface`  `Sylius\Plus\Inventory\Application\Updater\InventorySourceStockUpdater` has:

* The constructor of `Sylius\Plus\Inventory\Application\Updater\InventorySourceStockUpdater` has been changed
  from `public function __construct(FactoryInterface $inventorySourceStockFactory, RepositoryInterface $inventorySourceStockRepository, ObjectManager $objectManager)`
  to `public function __construct(FactoryInterface $inventorySourceStockFactory)`

* The constructor of `Sylius\Plus\Returns\Application\CommandHandler\ReturnUnitsToInventoryHandler` takes `RepositoryInterface $inventorySourceStockRepository` as the 3rd argument

* The return type of method `Sylius\Plus\Inventory\Application\Operator\ReturnInventoryOperator::returnUnitToInventorySource` has been changed from `void` to `?InventorySourceStockInterface`

# UPGRADE FROM 0.38.0 to 0.39.0

## General update
```bash
composer require "sylius/plus:0.39.*"
```

* Definition of route `sylius_plus_admin_shipment_split` has been changed, now it uses custom controller 
that dispatches `SplitAndSendShipment` command instead of using `ResourceController` with route configuration.

# UPGRADE FROM 0.37.0 to 0.38.0

## General update
```bash
composer require "sylius/plus:0.38.*"
```

* The key `shipmentId` has been changed to `shipment` in request body of endpoint `POST /admin/shipments`

# UPGRADE FROM 0.36.0 to 0.37.0

## General update
```bash
composer require "sylius/plus:0.37.*"
```

* `Sylius\Plus\Rbac\Application\Privilege\CompositePrivilege` class constructor parameters has been modified 
from `__construct(FilesystemCache $filesystemCache)` to `__construct(CacheInterface $cache)`

* `Sylius\Plus\Rbac\Infrastructure\Twig\RoutingExtension` class has become finalized and its constructor parameters has been modified 
from `__construct(UrlGeneratorInterface $generator, AdminPermissionResolverInterface $adminPermissionResolver)` 
to `__construct(AdminPermissionResolverInterface $adminPermissionResolver, BaseRoutingExtension $baseRoutingExtension)`

* `Sylius\Plus\Rbac\Infrastructure\Twig\HttpKernelExtension` and `Sylius\Plus\Rbac\Infrastructure\Twig\RoutingExtension` 
extend now `Twig\Extension\AbstractExtension` instead of `Symfony\Bridge\Twig\Extension\HttpKernelExtension`

# UPGRADE FROM 0.35.0 to 0.36.0

## General update
```bash
composer require "sylius/plus:0.36.*"
```

* You need to upgrade Sylius to v1.9.0.
    Please follow [Sylius's upgrade instructions](https://github.com/Sylius/Sylius/blob/master/UPGRADE-1.9.md).

* You need to upgrade RefundPlugin to v1.0.0-RC.8
  Please follow [RefundPlugin's upgrade instructions](https://github.com/Sylius/RefundPlugin/blob/master/UPGRADE.md#upgrade-from-100-rc7-to-100-rc8).

* Remove the `Resources/views/` part from templates paths, like in the following example: `@SyliusPlusPlugin/Resources/views/Grid/Field/_channels.html.twig` => `@SyliusPlusPlugin/Grid/Field/_channels.html.twig`

* `Sylius\Plus\Doctrine\ORM\OrderItemUnitRepositoryInterface` extends now `Sylius\Bundle\CoreBundle\Doctrine\ORM\OrderItemUnitRepository`

* `Sylius\Plus\Doctrine\ORM\OrderItemUnitRepository` extends now `Sylius\Component\Core\Repository\OrderItemUnitRepositoryInterface`

* Below classes are removed and replaced by other from Sylius:
  - `Sylius\Plus\Doctrine\ORM\OrderRepository` => `Sylius\Bundle\CoreBundle\Doctrine\ORM\OrderRepository`
  - `Sylius\Plus\Doctrine\ORM\OrderRepositoryInterface` => `Sylius\Bundle\CoreBundle\Doctrine\ORM\OrderRepositoryInterface`
  - `Sylius\Plus\DataProvider\OrderCollectionDataProvider` => `Sylius\Bundle\ApiBundle\DataProvider\OrderCollectionDataProvider`
  - `Sylius\Plus\DataTransformer\CommandAwareInputDataTransformer` => `Sylius\Bundle\ApiBundle\DataTransformer\CommandAwareInputDataTransformer`
  - `Sylius\Plus\Fixture\OptionsResolver\LazyOption` => `Sylius\Bundle\CoreBundle\Fixture\OptionsResolver\LazyOption`
  - `Sylius\Plus\Fixture\OptionsResolver\ResourceNotFoundException` => `Sylius\Bundle\CoreBundle\Fixture\OptionsResolver\ResourceNotFoundException` 

After these changes `Sylius Plus` will work with `Sylius 1.9` and `Symfony 4.4`, if you want to upgrade to `Symfony 5.2`, 
you need to upgrade `Sylius Plus` to next version (`0.37`).

# UPGRADE FROM 0.34.0 to 0.35.0

## General update
```bash
composer require "sylius/plus:0.35.*"
```

All versions including this and below are not compatible with `Sylius 1.9` out of the box. If this is your case,
please change the requirement in `composer.json` to: `"sylius/sylius": "~1.8.*"`

* You need to upgrade RefundPlugin to v1.0.0-RC.6
    Please follow [RefundPlugin's upgrade instructions](https://github.com/Sylius/RefundPlugin/blob/master/UPGRADE.md#upgrade-from-100-rc5-to-100-rc6).

* `Sylius\Plus\Doctrine\ORM\CreditMemoRepositoryInterface` extends now `Sylius\RefundPlugin\Doctrine\ORM\CreditMemoRepositoryInterface`
and `Sylius\Plus\Doctrine\ORM\CreditMemoRepository` extends now `Sylius\RefundPlugin\Doctrine\ORM\CreditMemoRepository`,
so if you extended these files, you should change the base classes.

# UPGRADE FROM 0.33.0 to 0.34.0

## General update
```bash
composer require "sylius/plus:0.34.*"
```

* `Sylius\Plus\Loyalty\Application\Command\BuyLoyaltyPurchase` class constructor parameters has been modified from `__construct(string $customerEmail, string $loyaltyPurchaseCode)` to `__construct(string $loyaltyPurchaseCode, ?string $customerEmail = null)`
* `Sylius\Plus\Loyalty\Infrastructure\DataTransformer\LoyaltyRuleActionDataTransformer` class has been replaced by `Sylius\Plus\Loyalty\Infrastructure\DataTransformer\LoyaltyRuleActionInputDataTransformer`

# UPGRADE FROM 0.32.0 to 0.33.0

## General update

```bash
composer require "sylius/plus:0.33.*"
```

# UPGRADE FROM 0.31.0 to 0.32.0

## General update

```bash
composer require "sylius/plus:0.32.*"
```

* `Sylius\Plus\Loyalty\Application\Provider\OrderLoyaltyPointsProvider` class has been removed and the logic from it has been moved to `Sylius\Plus\Entity\OrderTrait`
* `Sylius\Plus\Loyalty\Application\Provider\OrderLoyaltyPointsProviderInterface` class has been removed and the logic from it has been moved to `Sylius\Plus\Entity\OrderInterface`
* `Sylius\Plus\Loyalty\Infrastructure\Twig\OrderLoyaltyPointsExtension` class has been removed and the logic from it has been moved to `Sylius\Plus\Entity\OrderTrait`
Now you can using all logic from `OrderLoyaltyPointsExtension` directly on `Order` entity

# UPGRADE FROM 0.30.0 to 0.31.0

## General update

```bash
composer require "sylius/plus:0.31.*"
```

# UPGRADE FROM 0.29.0 to 0.30.0

## General update

```bash
composer require "sylius/plus:0.30.*"
```

* `Sylius\Plus\Doctrine\ORM\InvoiceRepositoryInterface` extends now `Sylius\InvoicingPlugin\Doctrine\ORM\InvoiceRepositoryInterface`.
* `Sylius\Plus\Doctrine\ORM\InvoiceRepository` extends now `Sylius\InvoicingPlugin\Doctrine\ORM\InvoiceRepository`.

# UPGRADE FROM 0.28.X to 0.29.0

## General update

```bash
composer require "sylius/plus:0.29.*"
```

* Update parameters and access control in your `config/packages/security.yaml` file:

    ```diff
        parameters:
            ...
    +       sylius.security.new_api_user_account_route: "%sylius.security.new_api_shop_route%/account"
    +       sylius.security.new_api_user_account_regex: "^%sylius.security.new_api_user_account_route%"

        security:
            access_control:
                ...
    +           - { path: "%sylius.security.new_api_user_account_regex%/.*", role: ROLE_USER }
    -           - { path: "%sylius.security.new_api_shop_regex%/.*", role: IS_AUTHENTICATED_ANONYMOUSLY }
                - { path: "%sylius.security.new_api_route%/shop/authentication-token", role: IS_AUTHENTICATED_ANONYMOUSLY }
    +           - { path: "%sylius.security.new_api_shop_regex%/.*", role: IS_AUTHENTICATED_ANONYMOUSLY }
     
# UPGRADE FROM 0.27.0 to 0.28.0

## General update

```bash
composer require "sylius/plus:0.28.*"
```

* You need to handle **Refund Plugin** migrations in a similar way to **Invoicing Plugin**.

    > TIP

    Take a look at `vendor/sylius/plus/etc/migrations-1.8-refund-plugin.sh` script - it would execute 
    marking all migrations automatically by:
    
    ```bash
    ./vendor/sylius/plus/etc/migrations-1.8-refund-plugin.sh
    ```

# UPGRADE FROM 0.26.0 to 0.27.0

## General update

```bash
composer require "sylius/plus:0.27.*"
```

# UPGRADE FROM 0.25.2 to 0.26.0

## General update

```bash
composer require "sylius/plus:0.26.*"
```

* You need to upgrade Sylius to version 1.8.1: https://github.com/Sylius/Sylius/blob/master/UPGRADE-1.8.md
Remember to place `Sylius\Plus\SyliusPlusPlugin` after `Sylius\Bundle\ApiBundle\SyliusApiBundle` in your `config/bundles.php`.

    > TIP

    Take a look at `vendor/sylius/plus/etc/migrations-1.8-invoicing-plugin.sh` and `vendor/sylius/plus/etc/migrations-1.8-plus.sh` 
    scripts - it would execute marking all migrations automatically by:
    
    ```bash
    ./vendor/sylius/plus/etc/migrations-1.8-invoicing-plugin.sh
    ./vendor/sylius/plus/etc/migrations-1.8-plus.sh
    ```

* In addition to the above UPGRADE file, mark all migrations from **@SyliusPlus** and **@SyliusInvoicingPlugin** as executed.

* Update security provider for `sylius_api_shop_user_provider` in your `config/packages/security.yaml` file: 
    ```diff
            security:
                providers:
                    ...
        -           sylius_api_shop_user_provider:
        -               id: sylius.shop_user_provider.email_or_name_based
        +           sylius_api_shop_user_provider:
        +               id: Sylius\Plus\CustomerPools\Infrastructure\Provider\UsernameAndCustomerPoolProvider
    ```

## Update templates

* Copy new templates that are overriden by Sylius Plus into `templates/bundles`:
    ```
    vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Customer/_form.html.twig
    ```

* Update file `templates/bundles/SyliusAdminBundle/Channel/_form.html.twig` by adding:

    ```diff
        ...
        <div class="ui labeled input">
    -       <div class="ui label">http://</div>
    +       <div class="ui label">https://</div>
        ...
    ```

* Update file `templates/bundles/SyliusAdminBundle/Order/Show/_header.html.twig` by adding:

    ```diff
        ...
        <div class="item">
    -       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|locale_name }}
    +       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|sylius_locale_name }}
        </div>
        ...
    ```

* Update file `templates/bundles/SyliusShopBundle/Account/Order/Show/_header.html.twig` by adding:

    ```diff
        ...
        <div class="item">
    -       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|locale_name }}
    +       {{ flags.fromLocaleCode(order.localeCode) }}{{ order.localeCode|sylius_locale_name }}
        </div>
        ...
    ```

* Update file `templates/bundles/SyliusUiBundle/Grid/_default.html.twig` by adding:

    ```diff
        ...
        <div class="title {% if criteria is not null %}active{% endif %}">
            <i class="dropdown icon"></i>
    +       <i class="filter icon"></i>
        ...
    ```
  
* The `Sylius\Plus\Inventory\Application\Checker\AvailabilityCheckerInterface` interface have brought back compatibility 
with `Sylius\Component\Inventory\Checker\AvailabilityCheckerInterface`. As a result, service definition of 
`Sylius\Plus\Inventory\Application\Checker\AvailabilityChecker` has been changed. The method 
`public function isStockSufficient(VariantsQuantityMapInterface $variantsQuantityMap): bool` has been moved to 
`Sylius\Plus\Inventory\Application\Checker\VariantQuantityMapAvailabilityChecker`, so service definition needs to be 
adjusted accordingly. What is more, the constructor of `AvailabilityChecker` has been adjusted as well. Services expected 
now are `Sylius\Component\Inventory\Checker\AvailabilityCheckerInterface` (for decoration purposes) and 
`Sylius\Plus\Inventory\Application\Checker\IsStockSufficientCheckerInterface`. 

# UPGRADE FROM 0.25.0 to 0.25.1

## General update

* `Sylius\Plus\Loyalty\Infrastructure\Form\DataTransformer\ChangeActionToChannelsBasedItemsTotalToPointsRatioConfigurationTransformer` class has been renamed to `Sylius\Plus\Loyalty\Infrastructure\Form\DataTransformer\ChangeActionToChannelsBasedItemsTotalToPointsRatioConfigurationTransformer` class.
* `Sylius\Plus\Loyalty\Infrastructure\Form\Type\Action\ChannelBasedItemsTotalToPointsRatioConfigurationType` class has been renamed to `Sylius\Plus\Loyalty\Infrastructure\Form\Type\Action\ChannelsBasedItemsTotalToPointsRatioConfigurationType` class.

# UPGRADE FROM 0.24.0 to 0.25.0

## General update

```bash
composer require "sylius/plus:0.25.*"
```

* Copy new templates that are overriden by Sylius Plus into `templates/bundles`:
    ```
    vendor/sylius/plus/src/Resources/templates/bundles/SyliusUiBundle/Security/_login.html.twig
    vendor/sylius/plus/src/Resources/templates/bundles/SyliusAdminBundle/Security/_content.html.twig
    ```

* `Sylius\Plus\Loyalty\Application\Calculator\PointsPerProductRatioCalculator` class has been replaced with `Sylius\Plus\Loyalty\Application\Calculator\ChannelsBasedPointsPerProductRatioCalculator` class.
* `Sylius\Plus\Loyalty\Infrastructure\Doctrine\ORM\EnabledLoyaltyPurchaseListQueryBuilder` class has been replaced with `Sylius\Plus\Loyalty\Infrastructure\Doctrine\ORM\ChannelRestrictingEnabledLoyaltyPurchaseListQueryBuilder` class.
* `Sylius\Plus\Loyalty\Infrastructure\Doctrine\ORM\EnabledLoyaltyPurchaseListQueryBuilderInterface` interface has been replaced with `Sylius\Plus\Loyalty\Infrastructure\Doctrine\ORM\ChannelRestrictingEnabledLoyaltyPurchaseListQueryBuilderInterface` interface.

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20200717100403.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200721091904.php src/Migrations/
```

# UPGRADE FROM 0.23.0 to 0.24.0

## General update

```bash
composer require "sylius/plus:0.24.*"
```

## Update templates

* `Sylius\Plus\Loyalty\Application\Processor\LoyaltyPointsProcessorInterface` interface has been removed.
* Constant types of loyalty rule actions has been moved:
    * from `Sylius\Plus\Loyalty\Domain\Rule\ItemsTotalToPointsRationRatio` to `Sylius\Plus\Loyalty\Application\Calculator\ChannelsBasedItemsTotalToPointsRatioCalculator`
    * from `Sylius\Plus\Loyalty\Domain\Rule\PointsPerProductRatio` to `Sylius\Plus\Loyalty\Application\Calculator\PointsPerProductRatioCalculator`
* `increaseBalance` and `decreaseBalance` methods in `Sylius\Plus\Loyalty\Domain\Model\LoyaltyPointsAccountInterface` have been removed in favour of `addTransaction` method.
* `value` field in `Sylius\Plus\Loyalty\Domain\Model\LoyaltyPurchase` has been changed to `loyaltyPoints`.

## Migrations

```bash
cp vendor/sylius/plus/migrations/Version20200702111344.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200706081938.php src/Migrations/
cp vendor/sylius/plus/migrations/Version20200709113414.php src/Migrations/
```

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
