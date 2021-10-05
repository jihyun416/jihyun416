---
layout: post
title:  NestJS - Interceptor
author: Jihyun
category: nestjs
tags:
- nestjs
date: 2021-10-05 15:40 +0900
---

### Interceptors

An interceptor is a class annotated with the `@Injectable()` decorator. Interceptors should implement the `NestInterceptor` interface.

인터셉터는 `@Injectable()` 데코레이터로 주석이 달린 클래스입니다. 인터셉터는 `NestInterceptor` 인터페이스를 구현해야 합니다.

![img](https://docs.nestjs.com/assets/Interceptors_1.png)

Interceptors have a set of useful capabilities which are inspired by the [Aspect Oriented Programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming) (AOP) technique. They make it possible to:

- bind extra logic before / after method execution
- transform the result returned from a function
- transform the exception thrown from a function
- extend the basic function behavior
- completely override a function depending on specific conditions (e.g., for caching purposes)

인터셉터에는 [AOP(Aspect Oriented Programming)](https://en.wikipedia.org/wiki/Aspect-oriented_programming) 기술에서 영감을 받은 유용한 기능 세트가 있습니다. 이를 통해 다음을 수행할 수 있습니다.

- 메소드 실행 전/후 추가 로직 바인딩
- 함수에서 반환된 결과를 변환
- 함수에서 던져진 예외 변환
- 기본 기능 동작 확장
- 특정 조건에 따라 기능을 완전히 재정의합니다(예: 캐싱 목적)



## Basics

Each interceptor implements the `intercept()` method, which takes two arguments. The first one is the `ExecutionContext` instance (exactly the same object as for [guards](https://docs.nestjs.com/guards)). The `ExecutionContext` inherits from `ArgumentsHost`. We saw `ArgumentsHost` before in the exception filters chapter. There, we saw that it's a wrapper around arguments that have been passed to the original handler, and contains different arguments arrays based on the type of the application. You can refer back to the [exception filters](https://docs.nestjs.com/exception-filters#arguments-host) for more on this topic.

각 인터셉터는 두개의 인수를 취하는 `intercept()` 메소드를 구현합니다. 첫번째는 `ExecutionContext` 인스턴스입니다 ([가드](https://docs.nestjs.kr/guards)와 정확히 동일한 객체). `ExecutionContext`는 `ArgumentsHost`에서 상속됩니다. 앞서 예외필터 장에서 `ArgumentsHost`를 보았습니다. 여기에서 원래 핸들러에 전달된 인수를 둘러싼 래퍼이며 애플리케이션 유형에 따라 다른 인수 배열을 포함하고 있음을 알았습니다. 이 주제에 대한 자세한 내용은 [예외필터](https://docs.nestjs.kr/exception-filters#arguments-host)를 다시 참조할 수 있습니다.



## Execution context

By extending `ArgumentsHost`, `ExecutionContext` also adds several new helper methods that provide additional details about the current execution process. These details can be helpful in building more generic interceptors that can work across a broad set of controllers, methods, and execution contexts. Learn more about `ExecutionContext`[here](https://docs.nestjs.com/fundamentals/execution-context).

`ArgumentsHost`를 확장함으로써 `ExecutionContext`는 현재 실행 프로세스에 대한 추가 세부정보를 제공하는 몇가지 새로운 헬퍼 메서드도 추가합니다. 이러한 세부정보는 광범위한 컨트롤러, 메서드 및 실행 컨텍스트에서 작동할 수 있는 보다 일반적인 인터셉터를 빌드하는데 도움이 될 수 있습니다. [여기](https://docs.nestjs.kr/fundamentals/execution-context)에서 `ExecutionContext`에 대해 자세히 알아보세요.



## Call handler

The second argument is a `CallHandler`. The `CallHandler` interface implements the `handle()` method, which you can use to invoke the route handler method at some point in your interceptor. If you don't call the `handle()` method in your implementation of the `intercept()` method, the route handler method won't be executed at all.

This approach means that the `intercept()` method effectively **wraps** the request/response stream. As a result, you may implement custom logic **both before and after** the execution of the final route handler. It's clear that you can write code in your `intercept()` method that executes **before** calling `handle()`, but how do you affect what happens afterward? Because the `handle()` method returns an `Observable`, we can use powerful [RxJS](https://github.com/ReactiveX/rxjs) operators to further manipulate the response. Using Aspect Oriented Programming terminology, the invocation of the route handler (i.e., calling `handle()`) is called a [Pointcut](https://en.wikipedia.org/wiki/Pointcut), indicating that it's the point at which our additional logic is inserted.

Consider, for example, an incoming `POST /cats` request. This request is destined for the `create()` handler defined inside the `CatsController`. If an interceptor which does not call the `handle()` method is called anywhere along the way, the `create()` method won't be executed. Once `handle()` is called (and its `Observable` has been returned), the `create()` handler will be triggered. And once the response stream is received via the `Observable`, additional operations can be performed on the stream, and a final result returned to the caller.

두번째 인수는 `CallHandler`입니다. `CallHandler` 인터페이스는 인터셉터의 특정지점에서 라우트 핸들러 메서드를 호출하는데 사용할 수 있는 `handle()` 메서드를 구현합니다. `intercept()` 메서드 구현에서 `handle()` 메서드를 호출하지 않으면 라우트 핸들러 메서드가 전혀 실행되지 않습니다.

이 접근방식은 `intercept()` 메서드가 요청/응답 스트림을 효과적으로 **포장**합니다. 결과적으로 최종 라우트 핸들러 실행 **전과 후에** 커스텀 로직을 구현할 수 있습니다. `handle()`을 호출하기 **전에** 실행되는 `intercept()` 메서드에 코드를 작성할 수 있다는 것은 분명하지만 이후에 일어나는 일에 어떤 영향을 미칠까요? `handle()` 메서드는 `Observable`을 반환하기 때문에 강력한 [RxJS](https://github.com/ReactiveX/rxjs) 연산자를 사용하여 응답을 추가로 조작할 수 있습니다. AOP(Aspect Oriented Programming) 용어를 사용하여 라우트 핸들러의 호출(즉, `handle()`호출)을 [Pointcut](https://en.wikipedia.org/wiki/Pointcut)이라고 하며, 추가 로직이 삽입됩니다.

예를 들어 들어오는 `POST /cats` 요청을 고려하십시오. 이 요청은 `CatsController` 내에 정의된 `create()` 핸들러를 대상으로 합니다. `handle()` 메서드를 호출하지 않는 인터셉터가 도중에 호출되면 `create()` 메서드가 실행되지 않습니다. `handle()`이 호출되면 (그리고 `Observable`이 반환되면) `create()` 핸들러가 트리거됩니다. 그리고 응답스트림이 `Observable`을 통해 수신되면 스트림에서 추가작업을 수행할 수 있으며 최종 결과가 호출자에게 반환됩니다.



## Aspect interception

The first use case we'll look at is to use an interceptor to log user interaction (e.g., storing user calls, asynchronously dispatching events or calculating a timestamp). We show a simple `LoggingInterceptor` below:

우리가 살펴볼 첫번째 사용사례는 인터셉터를 사용하여 사용자 상호작용을 기록하는 것입니다 (예: 사용자 호출저장, 이벤트를 비동기적으로 전달하거나 타임스탬프 계산). 아래에 간단한 `LoggingInterceptor`가 표시됩니다.

### logging.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

> **HINT**
>
> The `NestInterceptor<T, R>` is a generic interface in which `T` indicates the type of an `Observable<T>` (supporting the response stream), and `R` is the type of the value wrapped by `Observable<R>`.
>
> `NestInterceptor<T, R>`는 `T`가 `Observable<T>`(응답 스트림 지원)의 타입을 나타내고 `R`은 `Observable<R>`로 래핑된 값입니다.

> **NOTICE**
>
> Interceptors, like controllers, providers, guards, and so on, can **inject dependencies** through their `constructor`.
>
> 컨트롤러, 프로바이더, 가드 등과 같은 인터셉터는 `constuctor`를 통해 **종속성을 주입**할 수 있습니다.

Since `handle()` returns an RxJS `Observable`, we have a wide choice of operators we can use to manipulate the stream. In the example above, we used the `tap()` operator, which invokes our anonymous logging function upon graceful or exceptional termination of the observable stream, but doesn't otherwise interfere with the response cycle.

`handle()`은 RxJS `Observable`을 반환하므로 스트림을 조작하는 데 사용할 수 있는 다양한 연산자를 선택할 수 있습니다. 위의 예에서 우리는 관찰 가능한 스트림이 우아하거나 예외적으로 종료될 때 익명 로깅 함수를 호출하지만 응답주기를 방해하지 않는 `tap()` 연산자를 사용했습니다.



## Binding interceptors

In order to set up the interceptor, we use the `@UseInterceptors()` decorator imported from the `@nestjs/common` package. Like [pipes](https://docs.nestjs.com/pipes) and [guards](https://docs.nestjs.com/guards), interceptors can be controller-scoped, method-scoped, or global-scoped.

인터셉터를 설정하기 위해 `@nestjs/common` 패키지에서 가져온 `@UseInterceptors()` 데코레이터를 사용합니다. [파이프](https://docs.nestjs.kr/pipes) 및 [가드](https://docs.nestjs.kr/guards)와 마찬가지로 인터셉터는 컨트롤러 범위, 메서드 범위 또는 전역범위일 수 있습니다.

### cats.controller.ts

```typescript
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

> **HINT**
>
> The `@UseInterceptors()` decorator is imported from the `@nestjs/common` package.
>
> `@UseInterceptors()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

Using the above construction, each route handler defined in `CatsController` will use `LoggingInterceptor`. When someone calls the `GET /cats` endpoint, you'll see the following output in your standard output:

위의 구성을 사용하여 `CatsController`에 정의된 각 라우트 핸들러는 `LoggingInterceptor`를 사용합니다. 누군가 `GET /cats` 엔드포인트를 호출하면 표준출력에 다음과 같은 출력이 표시됩니다.

```typescript
Before...
After... 1ms
```

Note that we passed the `LoggingInterceptor` type (instead of an instance), leaving responsibility for instantiation to the framework and enabling dependency injection. As with pipes, guards, and exception filters, we can also pass an in-place instance:

인스턴스 대신 `LoggingInterceptor` 타입을 전달하여 인스턴스화를 프레임워크에 맡기고 종속성 주입을 활성화했습니다. 파이프, 가드, 예외필터와 마찬가지로 내부 인스턴스도 전달할 수 있습니다.

### cats.controller.ts

```typescript
@UseInterceptors(new LoggingInterceptor())
export class CatsController {}
```

As mentioned, the construction above attaches the interceptor to every handler declared by this controller. If we want to restrict the interceptor's scope to a single method, we simply apply the decorator at the **method level**.

In order to set up a global interceptor, we use the `useGlobalInterceptors()` method of the Nest application instance:

언급했듯이 위의 구성은 이 컨트롤러가 선언한 모든 핸들러에 인터셉터를 연결합니다. 인터셉터의 범위를 단일 메서드로 제한하려면 **메서드 수준**에서 데코레이터를 적용하면 됩니다.

전역 인터셉터를 설정하기 위해 Nest 애플리케이션 인스턴스의 `useGlobalInterceptors()` 메서드를 사용합니다.

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

Global interceptors are used across the whole application, for every controller and every route handler. In terms of dependency injection, global interceptors registered from outside of any module (with `useGlobalInterceptors()`, as in the example above) cannot inject dependencies since this is done outside the context of any module. In order to solve this issue, you can set up an interceptor **directly from any module** using the following construction:

글로벌 인터셉터는 모든 컨트롤러와 모든 라우트 핸들러에 대해 전체 애플리케이션에서 사용됩니다. 의존성 주입과 관련하여 모듈 외부에서 등록된 전역 인터셉터(위의 예에서와 같이 `useGlobalInterceptors()` 사용)는 모듈의 컨텍스트 외부에서 수행되므로 종속성을 주입할 수 없습니다. 이 문제를 해결하기 위해 다음 구성을 사용하여 **모든 모듈에서 직접** 인터셉터를 설정할 수 있습니다.

### app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

> **HINT**
>
> When using this approach to perform dependency injection for the interceptor, note that regardless of the module where this construction is employed, the interceptor is, in fact, global. Where should this be done? Choose the module where the interceptor (`LoggingInterceptor` in the example above) is defined. Also, `useClass` is not the only way of dealing with custom provider registration. Learn more [here](https://docs.nestjs.com/fundamentals/custom-providers).
>
> 이 접근방식을 사용하여 인터셉터에 대한 종속성 주입을 수행할 때 이 구성이 사용되는 모듈에 관계없이 인터셉터는 실제로 전역적입니다. 어디에서 해야 합니까? 인터셉터(위의 예에서는 `LoggingInterceptor`)가 정의된 모듈을 선택합니다. 또한 `useClass`가 커스텀 프로바이더 등록을 처리하는 유일한 방법은 아닙니다. [여기](https://docs.nestjs.kr/fundamentals/custom-providers)에서 자세히 알아보세요.



## Response mapping

We already know that `handle()` returns an `Observable`. The stream contains the value **returned** from the route handler, and thus we can easily mutate it using RxJS's `map()` operator.

우리는 이미 `handle()`이 `Observable`을 반환한다는 것을 알고 있습니다. 스트림에는 라우트 핸들러에서 **반환된** 값이 포함되어 있으므로 RxJS의 `map()` 연산자를 사용하여 쉽게 변경할 수 있습니다.

> **WARNING**
>
> The response mapping feature doesn't work with the library-specific response strategy (using the `@Res()` object directly is forbidden).
>
> 응답 매핑 기능은 라이브러리별 응답 전략에서 작동하지 않습니다 (`@Res()` 객체를 직접 사용하는 것은 금지됨).

Let's create the `TransformInterceptor`, which will modify each response in a trivial way to demonstrate the process. It will use RxJS's `map()` operator to assign the response object to the `data` property of a newly created object, returning the new object to the client.

프로세스를 보여주는 간단한 방법으로 각 응답을 수정하는 `TransformInterceptor`를 만들어 보겠습니다. RxJS의 `map()` 연산자를 사용하여 새로 생성된 객체의 `data` 속성에 응답 객체를 할당하고 새 객체를 클라이언트에 반환합니다.

### transform.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

> **HINT**
>
> Nest interceptors work with both synchronous and asynchronous `intercept()` methods. You can simply switch the method to `async` if necessary.
>
> Nest 인터셉터는 동기 및 비동기 `intercept()` 메서드 모두에서 작동합니다. 필요한 경우 메서드를 `async`로 간단히 전환할 수 있습니다.

With the above construction, when someone calls the `GET /cats` endpoint, the response would look like the following (assuming that route handler returns an empty array `[]`):

위의 구성에서 누군가 `GET /cats` 엔드포인트를 호출하면 응답은 다음과 같습니다 (라우트 핸들러가 빈 배열 `[]`을 반환한다고 가정).

```json
{
  "data": []
}
```

Interceptors have great value in creating re-usable solutions to requirements that occur across an entire application. For example, imagine we need to transform each occurrence of a `null` value to an empty string `''`. We can do it using one line of code and bind the interceptor globally so that it will automatically be used by each registered handler.

인터셉터는 전체 애플리케이션에서 발생하는 요구사항에 대한 재사용 가능한 솔루션을 만드는데 큰 가치를 둡니다. 예를 들어, `null`값의 각 항목을 빈 문자열 `''`로 변환해야 한다고 가정해보십시오. 한줄의 코드를 사용하여 이를 수행하고 인터셉터를 전역적으로 바인딩하여 등록된 각 핸들러가 자동으로 사용하도록 할 수 있습니다.

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class ExcludeNullInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(map(value => value === null ? '' : value ));
  }
}
```



## Exception mapping

Another interesting use-case is to take advantage of RxJS's `catchError()` operator to override thrown exceptions:

또 다른 흥미로운 사용사례는 RxJS의 `catchError()` 연산자를 활용하여 던져진(throw) 예외를 재정의하는 것입니다.

#### errors.interceptor.ts

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  BadGatewayException,
  CallHandler,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(
        catchError(err => throwError(new BadGatewayException())),
      );
  }
}
```



## Stream overriding

There are several reasons why we may sometimes want to completely prevent calling the handler and return a different value instead. An obvious example is to implement a cache to improve response time. Let's take a look at a simple **cache interceptor** that returns its response from a cache. In a realistic example, we'd want to consider other factors like TTL, cache invalidation, cache size, etc., but that's beyond the scope of this discussion. Here we'll provide a basic example that demonstrates the main concept.

때때로 핸들러 호출을 완전히 방지하고 대신 다른 값을 반환하려는 몇가지 이유가 있습니다. 분명한 예는 응답시간을 개선하기 위해 캐시를 구현하는 것입니다. 캐시에서 응답을 반환하는 간단한 **캐시 인터셉터**를 살펴 보겠습니다. 현실적인 예에서는 TTL, 캐시 무효화, 캐시 크기 등과 같은 다른 요소를 고려하고 싶지만 이 논의의 범위를 벗어납니다. 여기서는 주요 개념을 보여주는 기본 예를 제공합니다.

### cache.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isCached = true;
    if (isCached) {
      return of([]);
    }
    return next.handle();
  }
}
```

Our `CacheInterceptor` has a hardcoded `isCached` variable and a hardcoded response `[]` as well. The key point to note is that we return a new stream here, created by the RxJS `of()` operator, therefore the route handler **won't be called** at all. When someone calls an endpoint that makes use of `CacheInterceptor`, the response (a hardcoded, empty array) will be returned immediately. In order to create a generic solution, you can take advantage of `Reflector` and create a custom decorator. The `Reflector` is well described in the [guards](https://docs.nestjs.com/guards) chapter.

`CacheInterceptor`에는 하드코딩된 `isCached` 변수와 하드코딩된 응답 `[]`도 있습니다. 주목해야할 요점은 RxJS `of()` 연산자에 의해 생성된 새 스트림을 여기에 반환하므로 라우트 핸들러가 **전혀 호출되지 않습니다**. 누군가 `CacheInterceptor`를 사용하는 엔드포인트를 호출하면 응답(하드코딩된 빈 배열)이 즉시 반환됩니다. 일반적인 솔루션을 만들기 위해 `Reflector`를 활용하고 맞춤 데코레이터를 만들 수 있습니다. `Reflector`는 [가드](https://docs.nestjs.kr/guards) 장에 잘 설명되어 있습니다.



## More operators

The possibility of manipulating the stream using RxJS operators gives us many capabilities. Let's consider another common use case. Imagine you would like to handle **timeouts** on route requests. When your endpoint doesn't return anything after a period of time, you want to terminate with an error response. The following construction enables this:

RxJS 연산자를 사용하여 스트림을 조작할 수 있는 가능성은 우리에게 많은 기능을 제공합니다. 또 다른 일반적인 사용사례를 살펴 보겠습니다. 라우트 요청에서 **시간 초과**를 처리하고 싶다고 가정해보십시오. 일정시간이 지나도 엔드포인트가 아무것도 반환하지 않으면 오류 응답으로 종료하려고 합니다. 다음 구성은 이를 가능하게합니다.

### timeout.interceptor.ts

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, RequestTimeoutException } from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(new RequestTimeoutException());
        }
        return throwError(err);
      }),
    );
  };
};
```

After 5 seconds, request processing will be canceled. You can also add custom logic before throwing `RequestTimeoutException` (e.g. release resources).

5초후 요청 처리가 취소됩니다. `RequestTimeoutException`을 발생시키기 전에 커스텀 로직을 추가할 수도 있습니다(예: 리소스 해제)



#### 출처

> https://docs.nestjs.com/interceptors
>
> https://docs.nestjs.kr/interceptors

