# About this presentation

- Started looking at nodejs in anger about 1,5 month ago.
- Compiled a list of topics outside the usual (dev focused) tutorials
- Not all of this is battletested in production
- Beginners, there should be enough for you
- Advanced, Gurus , please chime in, I'd love your feedback!

# Things you should avoid (period)
## avoid typos, missing commas, etc..

- use jshint - <http://www.jshint.com/docs/> in combination with grunt

## avoid using global space: (use var where you can)

- use ```var a = 'bla'``` ;

## avoid using eval, with, switch: (without defaults)
- use ```'use strict';``` or  : ```$ node --use_strict```

<http://bishankochher.blogspot.fr/2012/02/nodejs-with-is-evil.html>
## don't run npm as root:
- npm pre-install in package.json can be evil

## don't run as root:
- use process.setuid, gid - <http://blog.liftsecurity.io/post/37388272578/writing-secure-express-js-apps>

```http.createServer(app).listen(app.get('port'), function(){
 console.log("Express server listening on port " + app.get('port'));
 process.setgid(config.gid);
 process.setuid(config.uid);
});```

- or sudo in your start script
- or use authbind or similar for non-priviledged users

# Testing
- mocha for mocha-watch
- sinon for spies, mocks, stubs
- nock for HTTP testing
- zombie for testing 
- phantomjs
- passport-stub

- mocha, vows, expect, ...
- zombie - http testing
- for socketio (force new connection)

use a grunt setup

# Versioning & Packaging
- use [semver](http://semver.org) versioning in package.json

## Dependencies in package.json
- dependencies
- shrinkwrap (freeze your dependencies)
- optionalDependencies
- [peerDependencies: use for plugins](http://blog.nodejs.org/2013/02/07/peer-dependencies/)

## Packaging
- npm pack
- [bashpack](https://github.com/jedi4ever/bashpack) (turns a nodejs process in bash script)

# Error handling
<http://snmaynard.com/2012/12/21/node-error-handling/>
## Try/catch , but ...
- Try/catch for errors, but this will not catch async errors

<http://www.javascriptkit.com/javatutors/trycatch2.shtml>

## Return an Error object and not a string
- The fundamental benefit of Error objects is that they automatically keep track of where they were built and originated
- ```callback(new Error('my own error'))```
- <http://www.devthought.com/2011/12/22/a-string-is-not-an-error/>

## Better stacktraces
- [better-stack-traces](https://github.com/mjpizz/better-stack-traces)
- [LongJohn (eventemitters)](http://www.mattinsler.com/post/26396305882/announcing-longjohn-long-stack-traces-for-node-js)
- [Longer stacks in chai](https://npmjs.org/package/chai-stack)

## Listen for ALL errors
- ```emit('error')``` makes the process exit if no listener:
- listen to http, redis connections , express, socketIO, log handler, metrics handler

- return a callback on error (as otherwise it will continue)

## Use domains to contain errors
- But imagine , 1 url gives an eror, it will actually exit the complete server! Lots of CPU power waisted for a single error
- Therefore the concept of domains were introduced 
- <http://nodejs.org/api/domain.html>
- with it you can create per 'domain' exceptions that will emit an on('error') to handle them
- Domains & express: <https://github.com/mathrawka/express-domain-errors>
- But it seems connect [grabs express exception with a try catch in it's code](https://github.com/senchalabs/connect/blob/master/lib/proto.js#L160-L195)
- Stack overflow explanation: <http://stackoverflow.com/questions/16174664/nodejs-error-handling-with-domains-and-socket-io>
- Domains & Connect: <https://github.com/baryshev/connect-domain>
- [Example on how to use connect-domain](http://masashi-k.blogspot.be/2012/12/express3-global-error-handling-domain.html)
The fundamental benefit of Error objects is that they automatically keep track of where they were built and originated

## process.uncaughtException
- it's very common in the nodejs world [to just exist on the exception](http://dshaw.github.io/2012-05-jsday/#/10)
**Not sure if this still makes sense with domains**

## return on a callback

    function(err,callback) {
      if (err) { return callback('err'); }
      console.log('this will be printed too')
    };

but: in loops or others ``returns`` means something else

## callback(err) vs emit('error')
**Not sure yet here**, both? only if callback, only if error listener?
- <https://groups.google.com/forum/m/#!topic/nodejs/QZa6bookqL0>
- <https://groups.google.com/forum/m/#!topic/nodejs/hqO0w6XgOlA>

# Logging
I like <https://github.com/flatiron/winston/>

- Has the logger.log, logger.info, logger.debug etc.
- Can separate multiple ``tags`` : <https://github.com/jedi4ever/socialapp/blob/master/lib/utils/logger.js>
- Create as a singleton, require and reuse in other modules (module cache will reuse same object)

  var logger = function (options) {

  // If we have already been initialized
  if (sharedLogger) {
    return sharedLogger;
  }


```var logger = require('../utils/logger')().loggers.get('express'); ```

## Multiple Outputs
- Console, File logging, rotation

- Also has a logstash output:
  - TCP: <https://github.com/jaakkos/winston-logstash>
  - UDP: <https://gist.github.com/mbrevoort/5848179>

## Use in express

    // enable web server logging; pipe those log messages through winston
    // http://stackoverflow.com/questions/9141358/how-do-i-output-connect-expresss-logger-output-to-winston
    var winstonStream = {
      write: function(message, encoding){
        logger.info(message.slice(0,-1)); //remove newline
      }
    };
    expressApp.use(express.logger({stream: winstonStream}));

## Use in socketio

``` var ioServer = socketIO.listen(webServer, { logger: logger , log:true});```

# Clustering
## The basics
Have multiple nodejs processes listen on the same socket.

The trick? pass the socket/file descriptor from a parent process and have the server.listen reuse that descriptor.
So **multiprocess** in their own memory space (but with ENV shared usually)

It does not balance, it leaves it to the kernel.

In the last nodejs > 0.8 there is a cluster module (functional although marked experimental)
- <http://nodejs.org/api/cluster.html>
- Simple cluster example: <https://gist.github.com/dsibilly/2992412>
- Simple cluster example + domains: <http://shapeshed.com/uncaught-exceptions-in-node/>
- Isaacs gist that was used as inspiration for the cluster doc - <https://gist.github.com/isaacs/5264418>

Note: not yet found a 100% reason to favor multi process/cluster nodejs over nginx/haproxy stuff:
- <http://blog.argteam.com/coding/hardening-node-js-for-production-part-3-zero-downtime-deployments-with-nginx/>
- Also see this [actionhero related blogpost on elegant downtime in relation to sockets , websockets etc..](http://blog.evantahler.com/blog/production-deployment-with-node-js-clusters.html)
- The rest off this post is to do clustering etc... yourself, otherwise you might want to check [actionhero.js](http://actionherojs.com/)

## Current Cluster/Domain Enhancements:
The core included module is basic, it tells you to take care of all the stuff you need in zerodowntime environments.
Most of the tools below allow you to:

- wait for workers to correctly close or sigKill if past timeout
- sigUSR2 to reload workers one by one
- some provide a cli to do the work using a socket/network connection
- kill a worker that has become unresponsive by waiting for a heartbeat
- put itself offline and not accepting any new requests

### Recluster: <https://github.com/doxout/recluster/>
- Code at: <https://github.com/doxout/recluster/blob/master/index.js>
- [Featured in this 10 steps to node Awesomeness](http://qzaidi.github.io/2013/05/14/node-in-production/)

- Good: Simple and in use
- Bad: No CLI , No domains, Cannot pass args

### Cluster2: <https://github.com/ql-io/cluster2>
- Framework in use at ebay: 

- Good: complete with monitoring, control URL
- Bad: Seems to be massive ...

### Naught: <https://github.com/superjoe30/naught> 
Note: naught2 was a temporary fork, but it got merged into master again

talks about Zero Downtime Crashed by intelligently handling express errors with domains
- <https://github.com/mathrawka/express-domain-errors>

- Good: simple, uses domains
- Bad: seems to do it's own logging

### cluster-master : <https://github.com/isaacs/cluster-master>
- Build by nodejs god @isaacs
- To be investigated

### Up: <https://github.com/LearnBoost/up>
- Seems to be the new learnbooost way for zero-downtime 
- <http://thechangelog.com/up-node-powered-zero-downtime-reloads-and-load-balancing/>

### Others to check:

- All Cluster npm modules: <https://npmjs.org/browse/keyword/cluster>
- Bowl: <https://github.com/waka/node-bowl>
- Herd: <https://github.com/segmentio/herd>
- Jumpstarter: <https://npmjs.org/package/jumpstarter>
- Multi Cluster: <https://npmjs.org/package/multi-cluster>
- Pluribus: <https://github.com/twistdigital/pluribus>
- Simple node cluster: <https://github.com/audreyt/node-cluster-server>

## Older solutions: 
(+2 years no updated & probably not nodejs > 0.8 compliant)
So you can safely ignore these, but they can give inspiration

- InfoQ blogpost on multi-core nodejs (is from 2010) - <http://www.infoq.com/articles/multi-core-node-js>
- **The inspiration:** has some cool options [but _alas < 0.8_ compliant](https://github.com/LearnBoost/cluster/issues/168) <http://learnboost.github.io/cluster/>
- <https://github.com/pgte/fugue/wiki/How-Fugue-Works>
- <https://github.com/kriszyp/multi-node> + 
[good writeup](http://www.sitepen.com/blog/2010/07/14/multi-node-concurrent-nodejs-http-server/)
- <https://github.com/dvv/stereo>

## Worth mentioning:
- <http://savanne.be/articles/deploying-node-js-with-systemd/>
- [Stackoverflow discussion Domains or forever](http://stackoverflow.com/questions/14611749/forever-or-domain-which-of-them-is-better-for-node-js-continuous-work#comment20409657_14614261)

# Profiling
**to be investigated**
- <http://naholyr.fr/2012/09/profiler-son-application-nodejs/>
- <http://mindon.github.io/blog/2012/04/26/profiling-nodejs-application/>

## Profiling tools
- <https://github.com/bnoordhuis/node-profiler>
- <http://nodetime.com/>
- <https://github.com/dannycoates/node-inspector>
- <https://github.com/dannycoates/v8-profiler>
- Callgrind - <http://valgrind.org/docs/manual/cl-manual.html>

- <http://blog.strongloop.com/announcing-a-new-and-improved-node-js-debugger/>
- <https://github.com/c4milo/node-webkit-agent>
- <https://github.com/devoidfury/express-debug>
- <http://badassjs.com/post/48702496345/tracegl-a-javascript-codeflow-visualization-and>
- <https://github.com/sidorares/node-tick>

## Connect
- Connect-profiler - <http://qzaidi.github.io/2012/07/15/node-profiling/>

##  Dtrace
- <https://github.com/chrisa/node-dtrace-provider#readme>
- <http://blog.nodejs.org/2012/04/25/profiling-node-js/>
- <https://github.com/bahamas10/node-dtrace-examples/tree/master/function-trace>
- <http://www.slideshare.net/bcantrill/instrumenting-the-realtime-web-nodejs-in-production>
- <http://s.urge.omniti.net/i/content/slides/Surge2012-DavidP_Nodejs.pdf>
- <https://npmjs.org/package/nodetrace>

# Memory
- <http://qzaidi.github.io/2012/08/05/pstack-for-nodejs/>
- <https://gist.github.com/qzaidi/3254449>

- Node-memwatch <https://hacks.mozilla.org/2012/11/tracking-down-memory-leaks-in-node-js-a-node-js-holiday-season/>
- <https://github.com/jedp/node-memwatch-demo>
- <http://code.osnap.us/wp/?p=43>
- <http://simonmcmanus.wordpress.com/2013/01/03/forcing-garbage-collection-with-node-js-and-v8/>

# 'Known' Limits/Tuning
- ulimit Filedescriptors
- Nagle algorithm
- # ports
- eventemittors/listeners
- Set listen backlog, max agents and max open files limit. -  <http://qzaidi.github.io/2013/05/14/node-in-production/>

- limit posts
- throttle

# Metrics
Use statsd backend to send counters, timers etc..

- [sivy/node-statsd](https://github.com/sivy/node-statsd/) is feature complete and seems to be the most popular 
- [dscape/lynx](https://github.com/dscape/lynx) has streams + (some wierd random/sampling stuff)
- [msiebuhr/node-statsd-client](https://github.com/msiebuhr/node-statsd-client) has express helpers & multi child options 
- [fasterize/node-statsd-profiler](https://github.com/fasterize/node-statsd-profiler) has transformation function
- [godmodelabs/statistik](https://github.com/godmodelabs/statistik) has a CLI interface

## <https://github.com/sivy/node-statsd/>

- reuse UDP connection: yes
- prefix: yes
- suffix: yes
- dnscache: yes
- mock: yes
- samplerate: yes
- errors: yes (eventemitter & bubbleup)

- timing: yes
- count: yes
- increment: yes
- decrement: yes
- gauge: yes
- set: yes

- batch: yes
- callback: yes

## <https://github.com/dscape/lynx>
- reuse UDP connection: yes (+ provide your own)
- prefix: yes (called scope)
- error:  provide an error function
- network : USE of ephemeral sockets!
- samplerate: yes (use special random)
- batching yes:

- increment: yes
- decrement: yes
- timing: yes
- gauge: yes
- set: yes

__special__:
- uses as stream in/out: yes (uses parser)

## <https://github.com/msiebuhr/node-statsd-client>
- reuse UDP connections: yes (ephemeral socket)
- prefix: yes

- count: yes
- gauges: yes
- increment: yes
- decrement: yes
- sets: yes
- timings: yes (delays)

__special__
- socketTimeout: yes
- children (multi prefix)
- express helper: yes (or per URL)

## <https://github.com/Singly/statsd-singly>

- **reuse UDP: no**
- callback: yes (but default prints to console)
- samplerate: yes
- **prefix: no**
- suffix: yes

- batch: yes
- count: yes
- gauges: yes
- increment: yes
- decrement: yes
- sets: yes (called modify)
- timings: (delays)

## <https://github.com/fasterize/node-statsd-profiler>
(fork from node-statsd)

- samplerate: yes
- timing: yes
- count: yes
- increment: yes
- decrement: yes
- gauge: yes
- set: ??

- timingstart: yes
- timignend: yes

__special__:
- introduces: key aliases
- transformKey function: YES

## <https://github.com/spreaker/nodejs-statsd-client>
Note: you can safely ignore this lib

- **reuse UDP connection: no**
- timing: yes
- count: yes
- increment: yes
- decrement: yes
- gauge: yes
- **errors: total ignore**
- **prefix: no**
- **samplerate: no**
- **callback: no**

## <https://github.com/godmodelabs/statistik>
Note: use node-statsd instead unless cli is something special for you. The feature set is smaller than node-statsd and no special features, so we'll ignore this

- udp connection reuse: no
- timing: yes
- counter: yes
- increment: yes
- decrement: yes
- gauge: yes
- rawSend: yes
- samplerate: yes
- **batch: no**

## Mixing in express & connect:
- <https://github.com/fetep/connect-logger-statsd/blob/master/lib/connect-logger-statsd.js>
Puts your responsetimes & status in statsd

- Although the fork of @sansmischevia seems to be more advanced https://github.com/fetep/connect-logger-statsd/network
Has ignore list, sends full path if needed, 

- <https://github.com/dokipen/connect-statsd/blob/master/index.js>
This focuses on writeHead, it will calculate the timeelapsed before sending back to the client

## Others:
Note: I've not included any specific backends here, we're focusing on generic statsd usage

- <https://github.com/dscape/winston-statsd> (logger -> statsd)

- <https://github.com/dscape/statsd-parser> (streaming parser)

- <https://github.com/Chatham/statsd-socket.io>
- <https://github.com/benjaminwootton/StatsdDashboard> (dashboard)

## What is a set in statsd?
Sets are acting like simple counters, with the additional specificity that it ignores duplicate values.

Technically, all values are stored in a set, and the number of elements in the set is sent to graphite during flushes. Sets are also emptied during flushes (in the same way that counters are reset to 0).

We have been using it in production for a while now, and it is working as expected. The use case that was used during the development of that feature was the following (it has been since extended to other cases as well):

    We want to graph the number of active logged in users on the website.
    Maintaining that state across application servers to manually update gauges is non-trivial.
    We send a message to statsd containing the id of the user making a request.

# CI
- Jenkins, Circle CI, TravisCI
- Chat bot in Campfire

# Configuration Mgmtm
- redis
- chef nodejs

# Continous delivery
- Use of forever <https://github.com/nodejitsu/forever> - just kidding

### Fleet: Extending the easy Git -> deploy
- Initial blogpost on fleet: <http://blog.nodejs.org/2012/05/02/multi-server-continuous-deployment-with-fleet/>
- Fleet - uses drones & propagit: <https://github.com/substack/fleet>
- Propagit: A cascading git deployment: <https://github.com/substack/propagit>
- blogpost on fleet usage: <http://opsite.wordpress.com/2013/05/04/automated-drone-management-system-for-node-js-fleet/>

- Flotilla: <https://npmjs.org/package/flotilla> 
- All nodejs fleet modules: <https://nodejsmodules.org/new/tags/fleet>
- <https://github.com/tblobaum/fleet-panel>
- <https://github.com/carlos8f/fleet>
- <https://github.com/nisaacson/fleet-stopall>
- <https://github.com/nisaacson/fleet-atc>
- <https://github.com/nisaacson/fleet-stopregex>
 
Not related, but also cool: EC2-fleet <https://github.com/ashtuchkin/ec2-fleet>

# Authz/Authn
- passport.js
- csrf in express <http://www.senchalabs.org/connect/middleware-csrf.html>
- helmet in express - <https://github.com/evilpacket/helmet>
- use bcrypt password - <http://codahale.com/how-to-safely-store-a-password/>
- link sessions socketio/express - <https://github.com/camarao/session.socket.io>

# Loadbalancer/SSL Termination

- <http://www.ericmartindale.com/2012/07/19/mitigating-the-beast-tls-attack-in-nodejs/>

- proxy in express
- in socketIO proxy
- HonorCiphers
- SSL offload via HAProxy 1.5dev (also websockets)
  brew install haproxy --devel
 - remove header express version
- Oauth
- CA options is an array
- SSL correct settings
- Perfect secrecy
- ```process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';```

- offload your SSL
- CA param is an array (add provider Certs)
- strictCipher, SSL attacks
- perfect secrecy
- SSL insecure!
- express proxy setting (X-...
- in socket.io (authentication Secure..) , Proxy

# Security stuff

## Various
- input/output checker

<http://www.slideshare.net/BishanSingh/node-security-the-good-bad-ugly>
console.log(with beep char);
JSON.parse
- Preinstall in npm
<http://www.slideshare.net/ASF-WS/asfws-2012-nodejs-security-old-vulnerabilities-in-new-dresses-par-sven-vetsch>
<https://www.google.be/url?sa=t&rct=j&q=&esrc=s&source=web&cd=14&ved=0CEQQFjADOAo&url=http%3A%2F%2Flab.cs.ttu.ee%2Fdl93&ei=WEndUYe4C8PKhAeztIHIDw&usg=AFQjCNE0MMCy8ZYdpi5O0gzr-2Qy5e2phg&sig2=6lFYPQ-FtKzKGHYoDcnL9Q>

- The request size is also not limited by Node.js which means that a large POST request can be sent to fill the whole memory. 

Mostly NPM is ran with root privileges.
Use objectProperties that cannot be changed
String sanitizer - <https://github.com/chriso/node-validator>

Fusker - fight back - <https://github.com/wearefractal/fusker>

<https://github.com/revington/connect-bruteforce>
<https://github.com/dharmafly/connect-ratelimit>

<http://stackoverflow.com/questions/14991963/socket-io-server-throttling-a-fast-client>

## Cookie/Sessions
- secure, http-only, signed cookies
- csrf attack connect.csrf
- helmet other security headers
- RedisStore backed
- connect-sessionIO
- redisstore with hiredis (reuse connection)
