# CSRF Protection
# CSRF 보호

- [Introduction](#csrf-introduction)
- [소개하기](#csrf-introduction)
- [Excluding URIs](#csrf-excluding-uris)
- [특정 URI 제외시키기](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Introduction
## 소개하기

Laravel makes it easy to protect your application from [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) attacks. Cross-site request forgeries are a type of malicious exploit whereby unauthorized commands are performed on behalf of an authenticated user.

라라벨은 [크로스-사이트 요청 위조 공격](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF)으로부터 당신의 애플리케이션을 쉽게 보호할 수 있게 해줍니다. 사이트 간 요청 위조는 인증된 사용자를 대신해서 승인되지 않은 커맨드를 악의적으로 활용하는 것입니다.

Laravel automatically generates a CSRF "token" for each active user session managed by the application. This token is used to verify that the authenticated user is the one actually making the requests to the application.

라라벨은 애플리케이션에 의해 관리되는 모든 활성화된 유저 세션마다 CSRF "토큰"을 자동으로 만듭니다. 이 토큰은 승인된 사용자가 애플리케이션에 요청을 할 수 있는 유일한 사람임을 확인하는데 사용됩니다.

Anytime you define a HTML form in your application, you should include a hidden CSRF token field in the form so that the CSRF protection middleware can validate the request. You may use the `csrf_field` helper to generate the token field:

HTML 폼을 정의할 때, CSRF 보호 미들웨어가 요청을 검증할 수 있도록 CSRF 토큰을 hidden 필드로 포함시켜야 합니다. 토큰 필드를 생성하기 위해 `csrf_field` 헬퍼를 사용할 수 있습니다.

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

The `VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), which is included in the `web` middleware group, will automatically verify that the token in the request input matches the token stored in the session.

`web` 미들웨어 그룹에 속한 `VerifyCsrfToken` [미들웨어](/docs/{{version}}/middleware)는 자동으로 요청에 포함된 토큰이 세션에 저장된 토큰과 일치하는지 확인할 것입니다.

#### CSRF Tokens & Vue
#### CSRF 토큰 & Vue

If you are using the [Vue](https://vuejs.org) JavaScript framework without the authentication scaffolding provided by the `make:auth` Artisan command, you will need to manually define a `Laravel` JavaScript object in your primary application layout.

This object specifies the CSRF token Vue should use when making requests:

`make:auth` 아티즌 명령어를 통한 인증 스캐폴딩 없이 [Vue](https://vuejs.org) 자바스크립트 프레임워크를 사용한다면, 기본 어플리케이션 레이아웃에 수동으로 `Laravel` 자바스크립트 객체를 정의해야 합니다. 이 객체는 request 가 있을 때 Vue 가 사용하는 CSRF 토큰을 지정합니다:

    <script>
        window.Laravel = {!! json_encode([
            'csrfToken' => csrf_token(),
        ]) !!};
    </script>

<a name="csrf-excluding-uris"></a>
## Excluding URIs From CSRF Protection
## CSRF 보호로 부터 특정 URI를 제외시키기

Sometimes you may wish to exclude a set of URIs from CSRF protection. For example, if you are using [Stripe](https://stripe.com) to process payments and are utilizing their webhook system, you will need to exclude your Stripe webhook handler route from CSRF protection since Stripe will not know what CSRF token to send to your routes.

URI를 CSRF 보호에서 제외시키고 싶을 때가 있습니다. 예를 들어서, 결제를 진행하기 위해  [Stripe](https://stripe.com)를 사용하고 있고, Stripe의 웹훅 시스템을 활용하고 있다면, Stripe 웹훅 핸들러 라우트를 CSRF 보호로 부터 제외시킬 필요가 있을 것입니다. 왜냐하면 Stripe 는 당신의 라우트에 어떤 CSRF 토큰을 보내야 하는지 모를 것이기 때문입니다.

Typically, you should place these kinds of routes outside of the `web` middleware group that the `RouteServiceProvider` applies to all routes in the `routes/web.php` file. However, you may also exclude the routes by adding their URIs to the `$except` property of the `VerifyCsrfToken` middleware:

일반적으로, 제외하고 싶은 라우트를 `RouteServiceProvie`가 `routes/web.php`에 있는 모든 라우트에 적용하는 `web` 미들웨어 그룹 외부에 두어야 합니다. 하지만, `VerifyCsrfToken` 미들웨어의 `$except` 속성에 URI를 추가하여 제외시킬 수도 있습니다.

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN
## X-CSRF-TOKEN

In addition to checking for the CSRF token as a POST parameter, the `VerifyCsrfToken` middleware will also check for the `X-CSRF-TOKEN` request header. You could, for example, store the token in a HTML `meta` tag:

POST 파라메터으로 넘어오는 CSRF 토큰을 체크하는 것에 더하여, `VerifyCsrfToken` 미들웨어는 `X-CSRF-TOKEN` 요청 헤더 역시 체크 할 것입니다. 예를들어, HTML `meta` 태그에 토큰을 저장할 수 있습니다.

    <meta name="csrf-token" content="{{ csrf_token() }}">

Then, once you have created the `meta` tag, you can instruct a library like jQuery to automatically add the token to all request headers. This provides simple, convenient CSRF protection for your AJAX based applications:

그리고나서, `meta` 태그를 만들고나면, jQeury와 같은 라이브러리를 사용하여 자동적으로 모든 헤더에 토큰을 추가할 수 있스빈다. 이러한 방식은 AJAX 기반 애플리케이션을 위한 간단하고 편리한 CSRF 보호 방법을 제공해줍니다.

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN
## X-XSRF-TOKEN

Laravel stores the current CSRF token in a `XSRF-TOKEN` cookie that is included with each response generated by the framework. You can use the cookie value to set the `X-XSRF-TOKEN` request header.

라라벨은 현재 CSRF 토큰을 프레임워크가 생성하는 모든 요청에 포함되어 있는 `XSRF-TOKEN` 쿠키에 저장합니다. `X-XSRF-TOKEN` 요청 헤더를 세팅하기 위해 쿠키 값을 사용할 수 있습니다.

This cookie is primarily sent as a convenience since some JavaScript frameworks, like Angular, automatically place its value in the `X-XSRF-TOKEN` header.

이 쿠키는 주로 편의를 위해 보내집니다. 왜냐하면 Angular 같은 자바스크립트 프레임워크는 그 값을 `X-XSRF-TOKEN` 헤더에 자동으로 설정하기 때문입니다.