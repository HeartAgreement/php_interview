#laravel - request一些方法
>请求方法的获取 / 检测请求方法
```
dd( $request->method() );
dd( $request->isMethod('post') );
//返回string get post delect put ...
```
>获取请求的路径 / 获取完整的url
```
dd( $request->path() )
// 例如localhost/ 是('/') 
// localhost/a 结果是('a')
// localhost/a/b 结果是('a/b')
// localhost/a/{b} 结果是('a/值')
dd( $request->url() );
//http://www.test.com/a/b
```
>获取请求的ip
```
string $request->ip() = array $request->getClientIp()[0]
array $request->getClientIps() = array $request->ips()
//这里是要注意代理的问题
/**
1.REMOTE_ADDR:浏览当前页面的用户计算机的ip地址 
2.HTTP_X_FORWARDED_FOR: 浏览当前页面的用户计算机的网关 
3.HTTP_CLIENT_IP:客户端的ip
为什么PHP里的HTTP_X_FORWARDED_FOR和Nginx的不一样
当你的网站使用了CDN后，用户会先访问CDN，
如果CDN没有缓存，则回源站（即你的反向代理）取数据。
CDN在回源站时，会先添加x_forwarded_for头信息，
保存用户的真实IP，而你的反向代理也会设定这个值，不过它不会覆盖，
而是把CDN服务器的IP（即当前remote_addr）添加到x_forwarded_for的后面，
这样x_forwarded_for里就会存在两个值。Nginx会使用这些值里的第一个，
即客户的真实IP，而PHP则会使用第二个，即CDN的地址。
为了能让PHP也使用第一个值，你需要添加以下fastcgi的配置。
fastcgi_param HTTP_X_FORWARDED_FOR $http_x_forwarded_for;
**/
```
1.没有使用代理服务器的情况
```
REMOTE_ADDR = 您的 IP 
HTTP_VIA = 没数值或不显示 
HTTP_X_FORWARDED_FOR = 没数值或不显示 
```
2.使用透明代理服务器的情 况：Transparent Proxies
```
REMOTE_ADDR = 最后一个代理服务器 IP 
HTTP_VIA = 代理服务器 IP 
HTTP_X_FORWARDED_FOR = 您的真实 IP ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。 
这类代理服务器还是将您的信息转发给您的访问对象，无法达到隐藏真实身份的目的。 
```
3.使用普通匿名代理服务器的情况：Anonymous Proxies 
```
REMOTE_ADDR = 最后一个代理服务器 IP 
HTTP_VIA = 代理服务器 IP 
HTTP_X_FORWARDED_FOR = 代理服务器 IP ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。 
隐藏了您的真实IP，但是向访问对象透露了您是使用代理服务器访问他们的。
```
4.使用欺骗性代理服务器的情况：Distorting Proxies 
```
REMOTE_ADDR = 代理服务器 IP 
HTTP_VIA = 代理服务器 IP 
HTTP_X_FORWARDED_FOR = 随机的 IP ，经过多个代理服务器时，这个值类似如下：
203.98.182.163, 203.98.182.163, 203.129.72.215。 
告诉了访问对象您使用了代理服务器，但编造了一个虚假的随机IP代替您的真实IP欺骗它。
```
5.使用高匿名代理服务器的情况：High Anonymity Proxies (Elite proxies) 
```
REMOTE_ADDR = 代理服务器 IP 
HTTP_VIA = 没数值或不显示 
HTTP_X_FORWARDED_FOR = 没数值或不显示 ，经过多个代理服务器时，这个值类似如下：203.98.182.163, 203.98.182.163, 203.129.72.215。 
完全用代理服务器的信息替代了您的所有信息，就象您就是完全使用那台代理服务器直接访问对象。 
```
##laravel获取ip源码
```
//这里的判断是非常简单 如果 REMOTE_ADDR地址在 HTTP_X_FORWARDED_FOR里就是有代理
//当有代理时源码都是使用getTrustedValues 这个方法获取ip地址的
    private static $trustedHeaders = array(
        self::HEADER_FORWARDED => 'FORWARDED',
        self::HEADER_X_FORWARDED_FOR => 'X_FORWARDED_FOR',
        self::HEADER_X_FORWARDED_HOST => 'X_FORWARDED_HOST',
        self::HEADER_X_FORWARDED_PROTO => 'X_FORWARDED_PROTO',
        self::HEADER_X_FORWARDED_PORT => 'X_FORWARDED_PORT',
    );
    private function getTrustedValues($type, $ip = null)
    {  
            $clientValues = array();
            $forwardedValues = array();
        // -1(11111) & 0b00010 (2) 恒等? && 是否存在 X_FORWARDED_HOST 
        // $clientValues[] = ( 0b00010 === 0b00010 恒等 ? ) '0.0.0.0:192.168.1.1'
        if ((self::$trustedHeaderSet & $type) && $this->headers->has(self::$trustedHeaders[$type])) {
            foreach (explode(',', $this->headers->get(self::$trustedHeaders[$type])) as $v) {
                $clientValues[] = (self::HEADER_X_FORWARDED_PORT === $type ? '0.0.0.0:' : '').trim($v);
            }
        }
        //
        if ((self::$trustedHeaderSet & self::HEADER_FORWARDED) && $this->headers->has(self::$trustedHeaders[self::HEADER_FORWARDED])) {
            $forwardedValues = $this->headers->get(self::$trustedHeaders[self::HEADER_FORWARDED]);
            $forwardedValues = preg_match_all(sprintf('{(?:%s)=(?:"?\[?)([a-zA-Z0-9\.:_\-/]*+)}', self::$forwardedParams[$type]), $forwardedValues, $matches) ? $matches[1] : array();
        }

        if (null !== $ip) {
            $clientValues = $this->normalizeAndFilterClientIps($clientValues, $ip);
            $forwardedValues = $this->normalizeAndFilterClientIps($forwardedValues, $ip);
        }

        if ($forwardedValues === $clientValues || !$clientValues) {
            return $forwardedValues;
        }

        if (!$forwardedValues) {
            return $clientValues;
        }

        if (!$this->isForwardedValid) {
            return null !== $ip ? array('0.0.0.0', $ip) : array();
        }
        $this->isForwardedValid = false;
        
        throw new ConflictingHeadersException(sprintf('The request has both a trusted "%s" header and a trusted "%s" header, conflicting with each other. You should either configure your proxy to remove one of them, or configure your project to distrust the offending one.', self::$trustedHeaders[self::HEADER_FORWARDED], self::$trustedHeaders[$type]));
    }
```
>获取端口
```
dd( $request->getPort() );
```
>获取请求头信息
```
$request->header('Connection')
```
```
$response->withCookie(cookie('cookie','learn-laravel',3)); 
//第一个参数是cookie名，第二个参数是cookie值，第三个参数是有效期（分钟）.
$response->withCookie(cookie()->forever('cookie-name','cookie-value')); 
//如果想要cookie长期有效使用此方法
```
