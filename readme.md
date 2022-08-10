# example redis-sentinel 

## use for yii2 

1) install package 
 `composer req jamescauwelier/psredis`

2) create interface

```php

namespace app\components\redis;

interface RedisSentinelInterface
{
}

```

3) add to application config 


```php
// main.php 
// configure DI
...
'definitions' => [
        RedisSentinelInterface::class => function (Container $container) {
            $sentinels = [
                [
                    // @TODO - FROM ENV
                    'h' => 'rerdis-sentinel',
                    'p' => 26379,
                ],
                [
                    'h' => 'rerdis-sentinel',
                    'p' => 26380,
                ],
                [
                    'h' => 'rerdis-sentinel',
                    'p' => 26381,
                
                ],
            ];
            $masterDiscovery = new MasterDiscovery('master-node');
            
            foreach ($sentinels as $sentinel) {
                $masterDiscovery->addSentinel(new Client($sentinel['h'], $sentinel['p']));
            }
            
            return new HAClient($masterDiscovery);
        },
    ],
    ...
    'components' => [
        'redis' => [
            'class' => RedisSentinelInterface::class,
        ],
    ],
```

```php

namespace app\components\auth;

use PSRedis\HAClient;
use Yii;

class JwtAuthHandler
{
    private const PREFIX_KEY = '_user_token_';
    
    public function handle(AuthEvent $event)
    {
        /** @var HAClient $redis */
        $redis = Yii::$app->redis;
        $token = $event->token;
        $jwtParams = Yii::$app->params['jwt'];
        $key = self::PREFIX_KEY . $token->getClaim('uid');
        $strToken = (string)$token;
        $redis->set($key, $strToken);
        // set expire and other operation
    }
}


```

# Важно

для доступа других контейнеров к редис нужно создать сеть и обозначить ее как external в docker-compose

```yml
...
networks:
  default:
    external:
      name: microservices_net
```

# Docker scale

`docker-compose up --detach --scale redis-master=1 --scale redis-replica=4 --scale redis-sentinel=3`
