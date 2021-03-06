Size.IO Proxy
==========

The Size.IO Proxy server is a high performance, lightweight and scalable method for collecting data locally to be relayed to the Size.IO platform in a fast and network-efficient manner with offline message queueing.  Send your messages to this proxy and let it worry about delivering them to the platform. It exposes several different interfaces supporting:

 * Redis clients in general
 * Plain TCP that is very Shell and one-liner friendly
 * Plain UDP for fast non-blocking messaging

Official documentation and code samples are available at **http://size.io/developer**

## Starting the proxy

The `size.js` is an executable Node.js script.  Install Node.js (http://nodejs.org/download/) on your system and then simply execute the script:

```bash
> ./size.js
29 Aug 00:53:32 - SERVER UDP Proxy bound to udp://*:6125
29 Aug 00:53:32 - SERVER TCP Proxy bound to tcp://*:6120
29 Aug 00:53:32 - SERVER Redis Proxy bound to tcp://*:6379
```

See below for information on how to use an upstart script to keep the service running in the background.

## Configuring the proxy

The `config.js` file contains configuration values worth reviewing.  Outside of setting a new access token, the default settings will work out of the box.

 * **exports.publisher_access_token** should be set to an Access Token with enough permissions to publish events via the Size.IO WebSocket interface.
 * **exports.subscriber_access_token** should be set to an Access Token with enough permissions to subscribe to events via the Size.IO WebSocket interface.
 * **exports.use_local_time** should be set to `false` unless you have a strong desire to assert the time of published events.
 * **exports.tcp.listen_ip** can be set to any desired IP address; leave it as `null` to listen on any address.
 * **exports.tcp.listen_port** can be set to any desired port. It defaults to `6120`
 * **exports.udp.listen_ip** can be set to any desired IP address; leave it as `null` to listen on any address.
 * **exports.udp.listen_port** can be set to any desired port. It defaults to `6125`
 * **exports.redis.listen_ip** can be set to any desired IP address; leave it as `null` to listen on any address.
 * **exports.redis.listen_port** can be set to any desired port. It defaults to `6379`
 * **exports.debug_level** can be set to any of the following values:
  * `0` marked as `DEBUG` in the output
  * `1` marked as `INFO` in the output
  * `2` marked as `WARN` in the output
  * `3` marked as `ERROR` in the output
  * Higher values will be marked `SERVER` in the log output

## Client examples

For event publishing, the different client interfaces exist to support software preferences and have no strong advantages over each other.  Only the Redis client interface supports subscriptions.

### PHP Redis Publish
```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
while (true) {
    $redis->incrby('api.get', 1);
	usleep(100000);
}
```

### PHP Redis Subscribe
```php
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
function echoEvent($redis, $channel, $message) {
	echo "'$channel' message: '$message'\n";
}
$redis->subscribe(array('event'), 'echoEvent');
```

### BASH TCP Publish
```bash
for i in {1..10}; do
    printf '{"k":"api.get","v":1}' | nc 127.0.0.1 6120
    sleep 0.1
done
```

### BASH UDP Publish
```bash
for i in {1..10}; do
    printf '{"k":"api.get","v":1}' | nc -u -w0 127.0.0.1 6125
    sleep 0.1
done
```

## Upstart script
An upstart script has been provided to automatically start the script on system start and to cleanly stop on system shutdown. Upstart will also restart the script should it crash for any reason.  Installation is easy:

 * Copy the `size-proxy.conf` to `/etc/init/`
 * Edit the script to set the `HOME` and `USER` variables, adapted for your sytem
 * Start the proxy with `sudo start size-proxy`
 * Stop the proxy with `sudo stop size-proxy`

Log files can be found in `/var/log/upstart/`.
