---
layout: post
title: "Enabling and configuring bundles from within AppKernel"
date: 2013-10-21 11:59
comments: true
categories: [Symfony]
---
I don't use Vagrant (I know, I know) and because of that, my develop machine (my laptop) does not have the same configuration as our server. And while I wanted to use [xhprof](https://github.com/facebook/xhprof) locally, our server didn't have it. I also wanted to use the [jns/xhprof-bundle](https://github.com/jonaswouters/XhprofBundle). 
<!-- more -->
So I threw the bundle in my composer.json file and decided I'd only load it for the `dev` environment *and* only if xhprof was installed. 

``` php AppKernel.php
if (in_array($this->getEnvironment(), array('dev', 'test'))) {
    // ...
    if (function_exists('xhprof_enable')) {
       $bundles[] = new Jns\Bundle\XhprofBundle\JnsXhprofBundle();
    }
}
```

Now, the issue with this is: the jns/xhprof-bundle requires some configuration. You can't just leave the configuration inside your config.yml, because then Symfony would complain that it doesn't know why those configuration values are there. 

I got the answer on the Symfony IRC channel from jrobeson: use a closure loader. Where you load your configuration file, add the following:
``` php AppKernel.php
public function registerContainerConfiguration(LoaderInterface $loader)
{
    $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');

    if (in_array($this->getEnvironment(), array('dev', 'test')) && function_exists('xhprof_enable')) {
        $closureLoader = function(ContainerBuilder $container) {
            $container->loadFromExtension('jns_xhprof', array(
                'location_lib'    => '/path/to/xhprof_lib.php',
                'location_runs'   => '/path/to/xhprof_runs.php',
                'location_config' => null,
                'location_web'    => 'http://localhost/xhprof/',
                'enabled'         => true,
            ));
        };
        $loader->load($closureLoader);
    }
}
```

**Note:** I know I could have used a different environment, but using this way, if xhprof was installed on the server, I could use it there as well (we have a staging environment on the server that runs the app in dev mode). 