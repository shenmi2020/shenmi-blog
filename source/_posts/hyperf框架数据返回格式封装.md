---
title: hyperf框架数据返回格式封装
date: 2022-01-21 11:18
categories: [编程]
tags:
 - [php]
 - [hyperf]
---

记录api成功和失败返回格式封装
<!-- more -->
### ResponseService

``` php
<?php

namespace App\Service\Utils;

use Hyperf\Di\Annotation\Inject;
use Hyperf\HttpServer\Contract\ResponseInterface;

/**
 * API请求返回服务类
 *
 * Class ResponseService
 *
 * @package App\Service\Utils
 */
class ResponseService
{
    /**
     * @var int
     */
    private $http_code = 200;

    /**
     * http头部信息
     *
     * @var string[]
     */
    private $http_headers = [
        'Author' => 'Colorado',
    ];

    /**
     * 业务返回码
     * 组成结构：xx(业务模块) xx(业务子模块) xx(细化编码)
     * 举例：110101 11(APP模块)01(鉴权模块)01(登录失败)
     *
     * @var int
     */
    private $business_code = 100000;

    /**
     * 业务返回消息
     *
     * @var string
     */
    private $business_msg = 'ok';

    /**
     * @Inject
     * @var ResponseInterface
     */
    private $response;

    /**
     * 设置http状态码
     *
     * @param int $code
     *
     * @return $this
     */
    public function setHttpCode(int $code = 200): self
    {
        $this->http_code = $code;

        return $this;
    }

    /**
     * 设置http头部信息
     *
     * @param string $name
     * @param mixed  $value
     *
     * @return $this
     */
    public function setHttpHeader(string $name, $value): self
    {
        $this->http_headers[$name] = (string)$value;

        return $this;
    }

    /**
     * 成功数据返回
     *
     * @param mixed $data           返回数据
     * @param int   $business_code  业务返回码
     *
     * @return ResponseInterface|\Psr\Http\Message\ResponseInterface
     */
    public function success($data, int $business_code = 100000)
    {
        $this->business_code = $business_code;

        return $this->response($data);
    }

    /**
     * 失败返回
     *
     * @param string $error_msg      错误信息
     * @param mixed  $data           返回数据
     * @param int    $business_code  错误业务码
     *
     * @return ResponseInterface|\Psr\Http\Message\ResponseInterface
     */
    public function fail(string $error_msg = 'fail', $data = null, int $business_code = 999999)
    {
        $this->business_code = $business_code;
        $this->business_msg = $error_msg;

        return $this->response($data);
    }

    /**
     * 返回数据
     *
     * @param $data
     *
     * @return ResponseInterface|\Psr\Http\Message\ResponseInterface
     */
    private function response($data)
    {
        $this->response = $this->response->json($this->normalizeData($data))->withStatus($this->http_code);

        if (! empty($this->http_headers)) {
            foreach ($this->http_headers as $name => $value) {
                $this->response = $this->response->withHeader($name, $value);
            }
        }

        return $this->response;
    }

    /**
     * 标准化返回数据格式
     *
     * @param mixed $data  业务返回数据
     *
     * @return array
     */
    private function normalizeData($data): array
    {
        return [
            'code'      => $this->business_code,
            'data'      => $data,
            'message'   => $this->business_msg,
            'timestamp' => time(),
        ];
    }
}

```

### AbstractController

``` php
public function __call($name, $arguments)
{
    if (method_exists(ResponseService::class, $name)) {
        return make(ResponseService::class)->{$name}(...$arguments);
    }
}
```

### Demo

``` php
// 成功
return $this->success($result);
```