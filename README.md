uptime
======

A simple remote monitoring utility using Node.js and MongoDB.

<img src="https://github.com/downloads/fzaninotto/uptime/uptime.png" title="Uptime screenshot" />

You can watch a [demo screencast on Vimeo](https://vimeo.com/39302164).

Features
--------

* Monitor thousands of websites (powered by [Node.js asynchronous programming](http://dotheweb.posterous.com/nodejs-for-php-programmers-1-event-driven-pro))
* Tweak frequency of monitoring on a per-check basis, up to the millisecond
* Receive instant web alerts on every page when a check goes down (thanks [socket.io](http://socket.io/))
* Record availability statistics for further reporting (powered by [MongoDB](http://www.mongodb.org/))
* Detailed uptime reports with animated charts (powered by [Highcharts](http://www.highcharts.com/))
* Monitor availability, responsiveness, average response time , and total uptime/downtime
* Get details about failed checks (HTTP error code, etc.)
* Group checks by tags and get reports by tag
* Familiar web interface (powered by [Twitter Bootstrap 2.0](http://twitter.github.com/bootstrap/index.html))
* complete API for integration with third-party monitoring services
* Easy installation and zero administration

Installing Uptime
-----------------------

As for every Node.js application, the installation is straightforward:

    > git clone git://github.com/fzaninotto/uptime.git
    > npm install
    > node app.js

Adding Checks
-------------

By default, the web UI runs on port 8082, so just browse to 

    http://localhost:8082/

And you're ready to begin. Create your first check by entering an URL, wait for the first ping, and you'll soon see data flowing through your charts!

Configuring
-----------

Uptime uses [node-config](https://github.com/lorenwest/node-config) to allow YAML configuration and environment support. Here is the default configuration, taken from `config/default.yaml`:

    mongodb:
      server:   localhost
      database: uptime
      user:     root 
      password:
    
    monitor:
      name:                   origin
      apiUrl:                 'http://localhost:8082/api'
      pollingInterval:        10000      # ten seconds
      updateInterval:         60000      # one minute
      qosAggregationInterval: 600000     # ten minutes
      timeout:                5000       # five seconds
      pingHistory:            8035200000 # three months
      http_proxy:      
    
    autoStartMonitor: true
    
    server:
      port:     8082

To modify this configuration, create a `development.yaml` or a `production.yaml` file in the same directory, and override just the settings you need. For instance, to run Uptime on port 80 in production, create a `production.yaml` file as follows:

    server:
      port:     80

Running The Monitor In a Separate Process
-----------------------------------------

Heavily browsing the web dashboard may slow down the server - including the polling monitor. In other terms, using the application can influence the uptime measurements. To avoid this effect, it is recommended to run the polling monitor in a separate process.

To that extent, set the `autoStartMonitor` setting to `false` in the `production.yaml`, and launch the monitor by hand:

    > node monitor.js &
    > node app.js

You can also run the monitor in a different server. This second server must be able to reach the API of the dashboard server: set the `monitor.apiUrl` setting accordingly in the `production.yaml` file of the monitor server.

You can even run several monitor servers in several datacenters to get average response time. In that case, make sure you set a different `monitor.name` setting for all monitor servers to be able to tell which server make a particular ping.

Using Plugins
-------------

Uptime provides plugins that you can enable to add more functionality.

To enable plugins, create a `plugins/index.js` module. This module must offer a public `init()` method, where you will require and initialize plugin modules. For instance, to enable only the `console` plugin:

    // in plugins/index.js
    exports.init = function() {
      require('./console').init();
    }

Currently supported plugins:

 * `console`: log pings and events in the console in real time

You can add your own plugins under the `plugins` directory. A plugin is simply a module with a public `init()` method. For instance, if you had to recreate a simple version of the `console` plugin, you could write it as follows:

    // in plugins/console/index.js
    var CheckEvent = require('../../models/checkEvent');
    exports.init = function() {
      CheckEvent.on('insert', function(checkEvent) {
        checkEvent.findCheck(function(err, check) {
          console.log(new Date() + check.name + checkEvent.isGoDown ? ' goes down' : ' goes back up');
        });
      });
    }

All Uptime entities emit lifecycle events that you can listen to on the Model class. These events are 'save', 'insert', 'update', and 'remove'.

License
-------

Uptime is free to use and distribute, under the MIT license. See the bundled `LICENSE` file for details.

If you like the software, please help improving it by contributing PRs on the [GitHub project](https://github.com/fzaninotto/uptime)!

TODO
----

* Allow email alerts in case of non-availability (not sure if this should be part of the lib)
* Account for scheduled maintenance (and provide two QoS calculations: with and without scheduled maintenance)
* Allow for JavaScript execution in the monitored resources by using a headless browser (probably zombie.js)
* Unit tests