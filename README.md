think-jwt for thinkphp5.1
=======
A simple library to encode and decode JSON Web Tokens (JWT) in PHP.

安装
------------

使用composer来管理依赖性并下载think-jwt:

```bash
composer require lishaoen/think-jwt
```

使用示例
-------
```php
<?php
use \lishaoen\JWT\JWT;

$key = "example_key";
$token = array(
    "iss" => "http://example.org",
    "aud" => "http://example.com",
    "iat" => 1356999524,
    "nbf" => 1357000000
);

$jwt = JWT::encode($token, $key);
$decoded = JWT::decode($jwt, $key, array('HS256'));

print_r($decoded);

$decoded_array = (array) $decoded;

/**
 * 在签名和验证服务器之间存在时钟偏差时，您可以添加一个余地来解释。建议这种余地不应超过几分钟。
 *
 */
JWT::$leeway = 60; // $leeway in seconds
$decoded = JWT::decode($jwt, $key, array('HS256'));

?>
```
Example with RS256 (openssl)
----------------------------
```php
<?php
use \Firebase\JWT\JWT;

$privateKey = <<<EOD
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQC8kGa1pSjbSYZVebtTRBLxBz5H4i2p/llLCrEeQhta5kaQu/Rn
vuER4W8oDH3+3iuIYW4VQAzyqFpwuzjkDI+17t5t0tyazyZ8JXw+KgXTxldMPEL9
5+qVhgXvwtihXC1c5oGbRlEDvDF6Sa53rcFVsYJ4ehde/zUxo6UvS7UrBQIDAQAB
AoGAb/MXV46XxCFRxNuB8LyAtmLDgi/xRnTAlMHjSACddwkyKem8//8eZtw9fzxz
bWZ/1/doQOuHBGYZU8aDzzj59FZ78dyzNFoF91hbvZKkg+6wGyd/LrGVEB+Xre0J
Nil0GReM2AHDNZUYRv+HYJPIOrB0CRczLQsgFJ8K6aAD6F0CQQDzbpjYdx10qgK1
cP59UHiHjPZYC0loEsk7s+hUmT3QHerAQJMZWC11Qrn2N+ybwwNblDKv+s5qgMQ5
5tNoQ9IfAkEAxkyffU6ythpg/H0Ixe1I2rd0GbF05biIzO/i77Det3n4YsJVlDck
ZkcvY3SK2iRIL4c9yY6hlIhs+K9wXTtGWwJBAO9Dskl48mO7woPR9uD22jDpNSwe
k90OMepTjzSvlhjbfuPN1IdhqvSJTDychRwn1kIJ7LQZgQ8fVz9OCFZ/6qMCQGOb
qaGwHmUK6xzpUbbacnYrIM6nLSkXgOAwv7XXCojvY614ILTK3iXiLBOxPu5Eu13k
eUz9sHyD6vkgZzjtxXECQAkp4Xerf5TGfQXGXhxIX52yH+N2LtujCdkQZjXAsGdm
B2zNzvrlgRmgBrklMTrMYgm1NPcW+bRLGcwgW2PTvNM=
-----END RSA PRIVATE KEY-----
EOD;

$publicKey = <<<EOD
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC8kGa1pSjbSYZVebtTRBLxBz5H
4i2p/llLCrEeQhta5kaQu/RnvuER4W8oDH3+3iuIYW4VQAzyqFpwuzjkDI+17t5t
0tyazyZ8JXw+KgXTxldMPEL95+qVhgXvwtihXC1c5oGbRlEDvDF6Sa53rcFVsYJ4
ehde/zUxo6UvS7UrBQIDAQAB
-----END PUBLIC KEY-----
EOD;

$token = array(
    "iss" => "example.org",
    "aud" => "example.com",
    "iat" => 1356999524,
    "nbf" => 1357000000
);

$jwt = JWT::encode($token, $privateKey, 'RS256');
echo "Encode:\n" . print_r($jwt, true) . "\n";

$decoded = JWT::decode($jwt, $publicKey, array('RS256'));

/*
 NOTE: This will now be an object instead of an associative array. To get
 an associative array, you will need to cast it as such:
*/

$decoded_array = (array) $decoded;
echo "Decode:\n" . print_r($decoded_array, true) . "\n";
?>
```


解释一下JWT
------------

>>JWT就是一个字符串，经过加密处理与校验处理的字符串，由三个部分组成。基于token的身份验证可以替代传统的cookie+session身份验证方法。三个部分分别如下：

#### header部分组成

header 格式为:
-----------------
```php
{
"typ":"JWT",
"alg":"HS256"
}

```
这就是一个json串，两个字段都是必须的,alg字段指定了生成signature的算法，默认值为 HS256,可以自己指定其他的加密算法，如RSA.经过base64encode就可以得到 header.

#### payload 部分组成

playload 基本组成部分：
--------------------------------

>>简单点：
```php
$payload=[
            'iss' => $issuer, //签发者
            'iat' => $_SERVER['REQUEST_TIME'], //什么时候签发的
            'exp' => $_SERVER['REQUEST_TIME'] + 7200 //过期时间
            'uid'=>1111
        ];
```

>> 复杂点：官方说法,三个部分组成(Reserved claims，Public claims,Private claims)
```php
$token   = [
            #非必须。issuer 请求实体，可以是发起请求的用户的信息，也可是jwt的签发者。
            "iss"       => "http://example.org",
            #非必须。issued at。 token创建时间，unix时间戳格式
            "iat"       => $_SERVER['REQUEST_TIME'],
            #非必须。expire 指定token的生命周期。unix时间戳格式
            "exp"       => $_SERVER['REQUEST_TIME'] + 7200,
            #非必须。接收该JWT的一方。
            "aud"       => "http://example.com",
            #非必须。该JWT所面向的用户
            "sub"       => "jrocket@example.com",
            # 非必须。not before。如果当前时间在nbf里的时间之前，则Token不被接受；一般都会留一些余地，比如几分钟。
            "nbf"       => 1357000000,
            # 非必须。JWT ID。针对当前token的唯一标识
            "jti"       => '222we',
            # 自定义字段
            "GivenName" => "Jonny",
            # 自定义字段
            "name"   => "Rocket",
            # 自定义字段
            "Email"     => "jrocket@example.com",
         
        ];
```
payload也是一个json数据，是表明用户身份的数据，可以自己自定义字段，很灵活。你也可以简单的使用，比如简单的方式。经过json_encode和base64_encode就可得到payload

#### signature组成部分

>> 将 header和 payload使用header中指定的加密算法加密，当然加密过程还需要自定秘钥，自己选一个字符串就可以了。
官网实例：
```php
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)

 自己使用：
  <?php

    public static function encode(array $payload, string $key, string $alg = 'SHA256')
    {
        $key = md5($key);
        $jwt = self::urlsafeB64Encode(json_encode(['typ' => 'JWT', 'alg' => $alg])) . '.' . self::urlsafeB64Encode(json_encode($payload));
        return $jwt . '.' . self::signature($jwt, $key, $alg);
    }

   public static function signature(string $input, string $key, string $alg)
    {
        return hash_hmac($alg, $input, $key);
    }
```

这三个部分使用.连接起来就是高大上的JWT,然后就可以使用了.


参考文章：
[https://www.cnblogs.com/xiekeli/p/5607107.html](https://www.cnblogs.com/xiekeli/p/5607107.html)
[https://segmentfault.com/a/1190000009981879](https://segmentfault.com/a/1190000009981879)
