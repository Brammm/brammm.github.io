---
layout: post
title: "Symfony: Absolute Asset URLs in Twig Templates"
date: 2013-08-12 07:30
comments: false
categories: [Development, Symfony2]
---

As with many applications, mine sends several e-mails. Some are sent instantaneously, some are sent from the spool. Whatever the case may be, you need absolute urls to links and assets. As an alternative to `path()`, Symfony provides the [`url()`](http://symfony.com/doc/current/book/templating.html#linking-to-pages) function for Twig, which creates an absolute url for a certain path. However, there's no alternative for `asset()`. The only solutions so far I've seen, required prepending `asset()` with `app.request.host` or something similar.
<!-- more -->
This is trivial when all emails are sent from within the browser and Symfony has an active request. However, when trying to send mails from the spool via command line, you'll be hit by an exception.

{% blockquote %}
You cannot create a service ("request") of an inactive scope ("request").
{% endblockquote %}

There's [a cookbook entry](http://symfony.com/doc/2.2/cookbook/console/sending_emails.html) that gives a hint, but not a solution. By setting the context parameters in your parameters.yml file, you can keep using the `url()` function in your Twig templates, but trying to use `app.request.host` (or similar) will still end with the same exception.

The simplest solution, in my opinion, is a Twig extension that will receive an instance of the Router:

``` php TwigExtension.php
namespace Acme\DemoBundle\Twig;

use Symfony\Component\Routing\RouterInterface;

class TwigExtension extends \Twig_Extension
{
    protected $router;

    /**
     * Receive Router instance
     */
    public function __construct(RouterInterface $router)
    {
        $this->router = $router;
    }

    /**
     * Declare the asset_url function
     */
    public function getFunctions()
    {
        return array(
            'asset_url' => new \Twig_Function_Method($this, 'assetUrl'),
        );
    }

    /**
     * Implement asset_url function
     * We get the router context. This will default to settings in
     * parameters.yml if there is no active request
     */
    public function assetUrl($path)
    {
        $context = $this->router->getContext();
        $host = $context->getScheme().'://'.$context->getHost();

        return $host.$path;
    }

    /**
     * Set a name for the extension
     */
    public function getName()
    {
        return 'twig_extension';
    }
}
```
Declare the extension in your services.yml and pass it the router service.
``` yml services.yml
services:
    twigextension:
        class:     Acme\DemoBundle\Twig\TwigExtension
        arguments: [@router]
        tags:
            - { name: twig.extension }
```
Make sure you don't forget the context parameters in parameters.yml
``` yml parameters.yml
parameters.yml
    router.request_context.host: example.com
    router.request_context.scheme: http
    router.request_context.base_url:
```
Now you can use this in your views:
{% raw %}
``` html
{{ asset_url('/path/to/asset.png') }}
```
{% endraw %}

And it will generate a url like "http://example.com/path/to/asset.png", doesn't matter if the view was generated during an actual request, or from command line.