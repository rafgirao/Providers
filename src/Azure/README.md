# Azure

```bash
composer require socialiteproviders/microsoft-azure
```

## Installation & Basic Usage

Please see the [Base Installation Guide](https://socialiteproviders.com/usage/), then follow the provider specific instructions below.

### Add configuration to `config/services.php`

```php
'azure' => [    
  'client_id' => env('AZURE_CLIENT_ID'),
  'client_secret' => env('AZURE_CLIENT_SECRET'),
  'redirect' => env('AZURE_REDIRECT_URI'),
  'tenant' => env('AZURE_TENANT_ID'),
  'proxy' => env('PROXY')  // optionally
],
```

### Add provider event listener

#### Laravel 11+

In Laravel 11, the default `EventServiceProvider` provider was removed. Instead, add the listener using the `listen` method on the `Event` facade, in your `AppServiceProvider` `boot` method.

* Note: You do not need to add anything for the built-in socialite providers unless you override them with your own providers.

```php
Event::listen(function (\SocialiteProviders\Manager\SocialiteWasCalled $event) {
    $event->extendSocialite('azure', \SocialiteProviders\Azure\Provider::class);
});
```
<details>
<summary>
Laravel 10 or below
</summary>
Configure the package's listener to listen for `SocialiteWasCalled` events.

Add the event to your `listen[]` array in `app/Providers/EventServiceProvider`. See the [Base Installation Guide](https://socialiteproviders.com/usage/) for detailed instructions.

```php
protected $listen = [
    \SocialiteProviders\Manager\SocialiteWasCalled::class => [
        // ... other providers
        \SocialiteProviders\Azure\AzureExtendSocialite::class.'@handle',
    ],
];
```
</details>

### Usage

You should now be able to use the provider like you would regularly use Socialite (assuming you have the facade installed):

```php
return Socialite::driver('azure')->redirect();
```

To logout of your app and Azure:
```php
public function logout(Request $request) 
{
     Auth::guard()->logout();
     $request->session()->flush();
     $azureLogoutUrl = Socialite::driver('azure')->getLogoutUrl(route('login'));
     return redirect($azureLogoutUrl);
}
```

### Returned User fields

- ``id``
- ``name``
- ``email``

## Advanced usage

In order to have multiple / different Active directories on Azure (i.e. multiple tenants) The same driver can be used but with a different config:

```php
/**
 * Returns a custom config for this specific Azure AD connection / directory
 * @return \SocialiteProviders\Manager\Config
 */
function getConfig(): \SocialiteProviders\Manager\Config
{
  return new \SocialiteProviders\Manager\Config(
    env('AD_CLIENT_ID', 'some-client-id'), // a different clientID for this separate Azure directory
    env('AD_CLIENT_SECRET'), // a different secret for this separate Azure directory
    url(env('AD_REDIRECT_PATH', '/azuread/callback')), // the redirect path i.e. a different callback to the other azureAD callbacks
    ['tenant' => env('AD_TENANT_ID', 'common')], // this could be something special if need be, but can also be left out entirely
  );
}
//....//
Socialite::driver('azure')
    ->setConfig(getConfig())
    ->redirect();
```

This also applies to the callback for getting the user credentials that one has to remember to inject the ```->setConfig($config)```-method i.e.:
```php
$socialUser = Socialite::driver('azure')
    ->setConfig(getConfig())
    ->user();
```

If the application that you are authenticating against is anything other single tenant, use the following values in place of the tenant_id:
 - Multitenant applications: "organizations"
 - Multitenant and personal accounts: "common"
 - Personal accounts only: "consumers"
