StaticPageBundle
================

## Installation

Install bundle through composer:
```
composer.phar require sokil/static-page-bundle
```

Add bundle to AppKernel:
```php
<?php

class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Cmf\Bundle\RoutingBundle\CmfRoutingBundle(),
            new Sokil\StaticPageBundle\StaticPageBundle(),
        );
    }
}
```

## Configuration

### Page view

Place file to `app/Resources/StaticPageBundle/views/Page/index.html.twig` with your own markup of static page.
Page instance `Sokil\StaticPageBundle\Entity\Page` accessable as `page` variable, also `locale` must be passed to template. To get localized data, call `page.getLocalizations()[locale]` which gives you instance of `Sokil\StaticPageBundle\Entity\PageLocalization`. 

### Routing

To enable default routing configuration just add `routing.yml` to you routes config `app/config/routing.yml`:
```yaml
static_page:
    resource: "@StaticPageBundle/Resources/config/routing.yml"
    prefix:   /
```
Or add your own routes for required actions.

To route any unexisted url to static page handler, you need to add some configuration to `app/config/config.yml`:
```yaml
cmf_routing:
    chain:
        routers_by_id:
            router.default: 200
            static_page.page_router: 100
    dynamic:
        persistence:
            orm:
                enabled: true
```

### Roles

Register role at `app/config/security.yml`:

```yaml
# http://symfony.com/doc/current/book/security.html
security:
    encoders:
        FOS\UserBundle\Model\UserInterface: sha512

    # http://symfony.com/doc/current/book/security.html#hierarchical-roles
    role_hierarchy:
        ROLE_PAGE_MANAGER:          [ROLE_USER]
```

## SPA view

Bundle has some SPA routes and dependencie for managing static pages, so you can configure them, 
for example with [sokil/frontend-bundle](https://github.com/sokil/FrontendBundle):

```html
{% import "@FrontendBundle/Resources/views/macro.html.twig" as frontend %}
{% import "@StaticPageBundle/Resources/views/macro.html.twig" as staticPageSpa %}

{{ staticPageSpa.jsResources() }}

<script type="text/javascript">
    (function() {
        // app options
        var options = {{ applicationData|json_encode|raw }};
        // router
        options.routers = [
            StaticPageRouter
        ];
        // requirejs
        options.requireJs = [
            StaticPageRequireJsConfig
        ];
        window.app = new Application(options);
        window.app.start();
    })();
</script>
```

### Deploy

If you want to use embedded editor, you need to setup SPA.

You need to execute grunt tasks to build [SPA](#SPA-view):
```
npm install
bower install
grunt
```

Bundle uses `assetic` bundle, so you need to register it in assetic config:
```yaml
assetic:
  bundles:
    - StaticPageBundle
```
