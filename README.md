# Shopify-App-Creation-Guide
Shopify App Creation Guide

```
 {{  -------------------------------------------------------------------------------------------------------------  }} 
 {{  -----------------       How To Create Laravel Shopify Apps using Osiset Package       -------------------   }}
 {   https://github.com/osiset/laravel-shopify/wiki/Creating-Webhooks   }
 {{  -------------------------------------------------------------------------------------------------------------  }} 
```

- 1: download laragon with php 7.4
- 2: composer create-project --prefer-dist laravel/laravel:^7.0 test_app
- 3: composer require osiset/laravel-shopify:16.* (  for Non-Embend App's )
  #                                                      OR
- 3: composer require osiset/laravel-shopify:17.* (  for Embend App's )
- 4: php artisan vendor:publish --tag=shopify-config
- 5: https://(your-domain).com/ ( Your app dir URL )
- 6: https://(your-domain).com/authenticate ( in app create section add Rediect's URL )
- 7: edit file ---> routes/web.php and modify the default route 
  ```
  Route::get('/', function () { return view('welcome'); })->middleware(['verify.shopify'])->name('home');
  ```
- 8: Modify file --->  resources/views/welcome.blade.php 

  ```
  @extends('shopify-app::layouts.default')
  @section('content')
      <!-- You are: (shop domain name) -->
      <p>You are: {{ $shopDomain ?? Auth::user()->name }}</p>
  @endsection
  @section('scripts')
      @parent
      <script>
          actions.TitleBar.create(app, { title: 'Welcome' });
      </script>
  @endsection
  ```
  
- 9: Edit file ---> app/User.php or app/Models/User.php
  
  ```
  use Osiset\ShopifyApp\Contracts\ShopModel as IShopModel;
  use Osiset\ShopifyApp\Traits\ShopModel;
  class User extends Authenticatable implements IShopModel
  use ShopModel;
  ```
  
- 10: For 16.* 
  ```
  package use --->  ( auth.shopify  )
  ```
#                           OR
- 10: For 17.* 
  ```
  package use --->  ( verify.shopify  )
  ```
  
- 11:  handle missing domainName exception  goto ---> app\Exceptions\Handler.php  & paste this code their
  
  ```
  public function render($request, Throwable $exception){
        if( $exception instanceof \Osiset\ShopifyApp\Exceptions\MissingShopDomainException ){
            return response()->view('login', [], 500);
        }
        return parent::render($request, $exception);
  }
  ```

- 12: Create login view ( login.blade.php ) 
  
  ```
  <form class="row g-4" action="{{ url('/authenticate') }}" method="GET">
     <div class="input-group mb-3">
         <input name="shop" type="text" class="form-control" placeholder="example.myshopify.com" aria-label="Recipient's username" aria-describedby="button-addon2">
         <button class="btn btn-outline-success" type="submit" id="button-addon2">Install</button>
      </div>
  </form>
  ```

- 13: For Non-embaded app do 
  ```
  'appbridge_enabled' => (bool) env('SHOPIFY_APPBRIDGE_ENABLED', false)
  ```
#                          OR
- 13: For Embaded app do 
```
'appbridge_enabled' => (bool) env('SHOPIFY_APPBRIDGE_ENABLED', true)
```

- 14: For creating webHooks  {  
  ```
  php artisan shopify-app:make:webhook [name] [topic]
  ```
  #              EXAMPLE ::-->   php artisan shopify-app:make:webhook OrdersCreateJob orders/create

- 15: After create webHook we have to config it in {   config/shopify-app.php  } File.
    LIKE :: -->  
    ```
      [
        'topic' => env('SHOPIFY_WEBHOOK_1_TOPIC', 'orders/create'),
        'address' => env('SHOPIFY_WEBHOOK_1_ADDRESS', 'https://some-app.com/webhook/orders-create')
      ],
      [
        'topic' => env('SHOPIFY_WEBHOOK_1_TOPIC', 'themes/publish'),
        'address' => env('SHOPIFY_WEBHOOK_1_ADDRESS', 'https://some-app.com/webhook/themes-publish')
      ],
     ```

- 16: Change it in .env file like this 
    LIKE :: -->  
    ```
      SHOPIFY_WEBHOOK_1_TOPIC=orders/create
      SHOPIFY_WEBHOOK_1_ADDRESS="${APP_URL}/webhook/orders-create"
      SHOPIFY_WEBHOOK_1_TOPIC=themes/publish
      SHOPIFY_WEBHOOK_1_ADDRESS="${APP_URL}/webhook/themes-publish"
    ```

- 17:  ADD  Shopify scopes in the scope in .env file  {  https://shopify.dev/api/usage/access-scopes   }
    LIKE :: -->  
                             
- 18: Create an web Route to clear cache  in database for webhooks    {  https://shopify.dev/api/admin-rest/2022-04/resources/webhook   }
    LIKE::--> 
    ```
       Route::get('/clear-cache', function() {
               Artisan::call('cache:clear');
               return "Cache is cleared";
       });
    ```
    
- 19:  Add Laravel logs Viewer package to view errors in detailed UI   {  https://github.com/R-jee/laravel-log-viewer  }
    LIKE::-->   
    ```
       Install via composer
                 composer require rap2hpoutre/laravel-log-viewer
       Add Service Provider to config/app.php in providers section
                 Rap2hpoutre\LaravelLogViewer\LaravelLogViewerServiceProvider::class,
       Add a route in your web routes file:
                 Route::get('logs', [\Rap2hpoutre\LaravelLogViewer\LogViewerController::class, 'index']);
    ```
    
- 20:    Log::info($input);

- 21:    php artisan make:model --migration --controller webhookJobs

- 22:  Create Charging Plans
  #     ---> create seeds in database section PlanSeeder.php
  ```
       <?php
          use Illuminate\Database\Seeder;
          use Osiset\ShopifyApp\Storage\Models\Plan;

          class PlanSeeder extends Seeder
          {
              /**
               * Seed the application's database.
               *
               * @return void
               */
              public function run()
              {
                  // $this->call(UsersTableSeeder::class);

                  /*
                   # Create a recurring "Demo" plan for $5.00, with 7 trial days, which will be presented on install to the shop and have the ability to issue usage charges to a maximum of $10.00
                      INSERT INTO plans (
                          `type`,
                          `name`,
                          `price`,
                          `interval`,
                          `capped_amount`,
                          `terms`,
                          `trial_days`,
                          `test`,
                          `on_install`,
                          `created_at`,
                          `updated_at`) VALUES
                      ('RECURRING/ONETIME','Test Plan',5.00,'EVERY_30_DAYS',10.00,'Test terms',7,FALSE,1,NULL,NULL);
                  */


                  $Plan = new Plan();
                  $Plan->type = "RECURRING";
                  $Plan->name = "Basic Plan";
                  $Plan->price = 4.99;
                  $Plan->interval = "EVERY_30_DAYS";
                  $Plan->capped_amount = 10.00;
                  $Plan->terms = "Basic Plan ~ amount 4.99";
                  $Plan->trial_days = 7;
                  $Plan->test = 1;
                  $Plan->on_install = 1;
                  $Plan->save();

                  $Plan = new Plan();
                  $Plan->type = "RECURRING";
                  $Plan->name = "Premiere Plan ~ amount 9.99";
                  $Plan->price = 9.99;
                  $Plan->interval = "EVERY_30_DAYS";
                  $Plan->capped_amount = 10.00;
                  $Plan->terms = "Premiere Plan ~ amount 9.99";
                  $Plan->trial_days = 14;
                  $Plan->test = 1;
                  $Plan->on_install = 1;
                  $Plan->save();

              }
          }
  ```

- 23 :  php artisan make:seeder PlanSeeder  ------->    php artisan db:seed

- 24 :  php artisan make:middleware CustomBillable

- 25:  
  ```
    <?php
      namespace App\Http\Middleware;
      use Closure;
      use Illuminate\Support\Facades\Config;
      class CustomBillable
      {
          /**
           * Handle an incoming request.
           *
           * @param  \Illuminate\Http\Request  $request
           * @param  \Closure  $next
           * @return mixed
           */
          public function handle($request, Closure $next)
          {
              info(json_encode($request));
              if( Config::get('shopify-app.billing_enabled') === true  ){
                  $shop = auth()->user();

                  if(!$shop->isFreemium() && !$shop->isGrandfathered() && $shop->plan == null ){
                  // if(!$shop->shopify_freemium && !$shop->shopify_grandfathered && $shop->plan == null ){
                      return redirect()->route('billing.plans');
                  }
              }
              return $next($request);
          }
      }
  ```
  
  # ADD  middleware in  Kernel.php file at  last 
  ```
    'custom.billable' => \Illuminate\Auth\Middleware\CustomBillable::class,
  ```
  
- 26:    
  ```
    <div class="bottom">
         <a href="{{ route('billing', ['plan' => $plans[0]->id ]) }}">Buy Now</a>
    </div>
    <div class="bottom">
         <a href="{{ route('billing', ['plan' => $plans[1]->id ]) }}">Buy Now</a>
    </div>

    <div class="bottom">
        <a href="{{ route('free.plan') }}">Get Access</a>
    </div>
  ```

- 27 :  
  ```
    Route::get('/free-plan', function(){
        User::where('id' , auth()->user()->id )->update(
            [
                'plan_id' => NULL,
                'shopify_freemium' => 1
            ]
        );
        return redirect()->route('home');
    })->name('free.plan');  
  ```
  
- 28:  App proxy in app
  ```
    Route::get('checkSetupStatus', 'CartButtonhiderDetailController@checkSetupStatus')->name('check.SetupStatus')->middleware(['auth.proxy']);
        public function checkSetupStatus(){
            $shop = User::where('name', request('shop'))->first();
            if($shop->status){
                return response()->json(['status' => true]);
            }else{
                return response()->json(['status' => false]);
            }
        }
  ```
  
# After Authentication Job -->
  - php artisan make:job AfterAuthenticateJob


 
