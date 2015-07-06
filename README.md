# IPLock
Simple webpage frequency cap or IP lock solution so you can protect your landing pages from snooping repeat visitors! 
Cloakers and Trackers have long implemented this feature. The most common (and awful) method of implementing this is using cookies. It really depends who you're trying to block but anyone worth blocking will have the sense to clear their cookies after each visit.

Here's a simple solution using PHP & Redis that blocks the actual IP with minimal impact on your page load time.

## Step 1: Get a Redis server
We'd recommend [compose.io](https://www.compose.io/redis/) for a hosted Redis solution. It's $18.50 per month for a 256mb node. This may seem like a lot but 256mb can go a long way if you consider that an IP string is worth 15 bytes. For example, even if we take 25% off our 256mb for overhead: 256,000,000 * 0.75 / 15 is 12 million IPs!

## Step 2: Download Predis 
[Download Predis](https://github.com/nrk/predis/zipball/v1.0.1), Unzip, copy the `src` directory to the same location as your landing page, renaming it to `Predis`

## Step 3: Landing page code
Place this code at the top of your landing page
```php
<?php
define('FREQ_CAP', 10);
define('FREQ_TIME', '1 day'); 

function capReached() {
    // output what you want here
    echo "Fake it till you make it";
    // or redirect using header('Location: http://anotherpage.com', true, 301);
    exit();
}

require 'Predis/Autoloader.php';
Predis\Autoloader::register();

// connect
$redis = new Predis\Client([
	'scheme'   => 'tcp',
	'host'     => 'YOUR_SERVER_HOST',
	'port'     => YOUR_SERVER_PORT_1234,
	'password' => 'YOUR_SERVER_PASSWORD'
]);

// perform check
$ip = $_SERVER['REMOTE_ADDR'];
$val = $redis->incr($ip);
if ($val == 1) 
    $redis->expireat($ip, strtotime('+'.FREQ_TIME));

if ($val > FREQ_CAP) 
    capReached();

```

##FAQ

###Is this free?

The code we posted here is free but the suggested compose.io hosted redis database costs just under $20 a month. So its not completely free to run. If you have a server with some spare capacity then of course you could run your own Redis database, but that is beyond the scope of this post.

###When does a visitor get blocked?

Lets assume FREQ_CAP=3 and FREQ_TIME=1 day. What this means is that visitors will get blocked after they have visited the page for the 3rd time within 1 day. 

The clock starts ticking on their first visit and the number of visits resets after FREQ_TIME is passed.

###How to redirect the blocked user

Simple. Change this function:

```
function capReached() {
    echo "Fake it till you make it";
    exit();
}
```

to something like this:

```
function capReached() {
    header('Location: http://anotherpage.com', true, 301);
    exit();
}
```
