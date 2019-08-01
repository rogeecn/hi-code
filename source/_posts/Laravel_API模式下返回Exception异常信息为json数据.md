---
title: Laravel API模式下返回Exception异常信息为json数据
date: 2018-05-21 00:00:00
tags: ["JSON"]
abbrlink: laravel-api-mo-shi-xia-fan-hui-exception-yi-chang-xin-xi-wei-json-shu-ju
img: ""
comments: false
---

当前应用为API应用时我们需要对返回的所有数据请求格式化为统一的 `JSON` 数据结构， 但是有些方法内抛出了 `exception` 时会导致页面收到HTML返回数据。
按下面的方法改下文件：`app/Exceptions/Handler.php`



```php
/**
 * Render an exception into an HTTP response.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Exception $e
 * @return \Illuminate\Http\Response
 */
public function render($request, Exception $e)
{
    // If the request wants JSON (AJAX doesn't always want JSON)
    if ($request->wantsJson()) {
        // Define the response
        $response = [
            'errors' => 'Sorry, something went wrong.'
        ];

        // If the app is in debug mode
        if (config('app.debug')) {
            // Add the exception class name, message and stack trace to response
            $response['exception'] = get_class($e); // Reflection might be better here
            $response['message'] = $e->getMessage();
            $response['trace'] = $e->getTrace();
        }

        // Default response of 400
        $status = 400;

        // If this exception is an instance of HttpException
        if ($this->isHttpException($e)) {
            // Grab the HTTP status code from the Exception
            $status = $e->getStatusCode();
        }

        // Return a JSON response with the response array and status code
        return response()->json($response, $status);
    }

    // Default to the parent class' implementation of handler
    return parent::render($request, $e);
}
```

这里仅仅是通过 `wantJson` 方法进行了判定，如果 `API` 请求者不带上 `Accept: application/json` 这个请求头的时候还是不会正确的返回JSON数据， 这时候可以加一个判定

```php
...
if ($request->wantJson() || $request->is("api/*"))
...
```

这里的 `api/*` 是应用 `api` 请求路径的根

完成。
