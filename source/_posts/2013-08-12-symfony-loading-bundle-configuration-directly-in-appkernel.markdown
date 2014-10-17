---
layout: post
title: "Symfony: Loading bundle configuration directly in AppKernel"
date: 2013-08-12 09:19
comments: off
categories: [Development, Symfony2]
---

I develop locally on my machine. On the server, I have a staging and production environment. Staging is run in dev, production in prod. Something you probably are familiar with. The server is monitored with [New Relic](http://newrelic.com/), but on my local machine, I wanted to use [XHProf](http://pecl.php.net/package/xhprof).
<!-- more -->
I wanted to use [jonaswouters' XHProfBundle](https://github.com/jonaswouters/XhprofBundle), but out of the box, this would have caused a bit of an issue when deploying: I couldn't simply enable the bundle in AppKernel.php because Staging also runs the dev environment and XHProf isn't installed on the server.

I decided to wrap the bundle loading in `if()`:
``` php AppKernel.php
if (in_array($this->getEnvironment(), array('dev', 'test'))) {
    $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
    $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
    $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
    if (function_exists('xhprof_enable')) {
        $bundles[] = new Jns\Bundle\XhprofBundle\JnsXhprofBundle();
    }
}
```
This would load the bundle only if XHProf was installed on the server.

However, the bundle needs a bit of configuration in config.yml. Trying to work with a configuration file that has configuration for bundles that aren't loaded will result in an InvalidArgumentException:
{% blockquote %}
There is no extension able to load the configuration for "jns_xhprof"
{% endblockquote %}

As I couldn't repeat the above `if()` in config.yml, I went looking for another solution. jrobeson over on [IRC](irc://irc.freenode.net/symfony) suggested a closure loader. In AppKernel.php, in `registerContainerConfiguration()` your config file is loaded using `$loader->load($resource)`. The resource being the path to your config file.

You can also pass a closure to the same method. That closure will receive an instance of `ContainerBuilder` which provides the method `loadFromExtension()`, which in turn can be used to load configuration from an array. This works like so:
``` php AppKernel.php
public function registerContainerConfiguration(LoaderInterface $loader)
{
    $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');

    $closureLoader = function($container) {
        $container->loadFromExtension('extension_name', array(
            // Config here
        ));
    };
    $loader->load($closureLoader);
}
```
*extension_name* is the name you would put in your config file, *jns_xhprof* in my case. All that's left to do was wrap the $closureLoader stuff in the same `if()` as the bundle loading and I was done. Now I can use XHProf locally and the server won't error on my next deploy.