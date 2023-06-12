# ZIO 速查表

- 该表基于 [ZIO](https://github.com/zio/zio) 2.0.X (编写时为 2.0.10).
- 为了简单起见，所有环境都被省略了，但是所有的函数都可以使用 `ZIO[R, E, A]` 的形式来使用。
- 为了简单起见，函数参数名称都被省略了。此外，函数的参数类型通常比下面显示的要“更通用”。
- 有一些函数对应的变体未在列表中提及，因为相关函数在概念上相似，只是在某些细节上有所不同。由于命名一致，通常可以举一反三。
- 除了基本效果类型 `ZIO[R, E, A]` 之外，其他的 ZIO
  类型都被省略了。例如：`ZStream[R, E, A]`, `ZLayer[RIn, E, ROut]`, `Fiber[E, A]` 和 `Ref[A]`。
- 在该表中，`E1` 代表 `E` 的派生，但 `E2` 可以是任何错误类型。同样适用于 `A` 和 `A1`。

## 别名

| 别名           | 全名                       |
|--------------|--------------------------|
| `UIO[A]`     | `ZIO[Any, Nothing, A]`   |
| `IO[E, A]`   | `ZIO[Any, E, A]`         |
| `Task[A]`    | `ZIO[Any, Throwable, A]` |
| `RIO[R, A]`  | `ZIO[R, Throwable, A]`   |
| `URIO[R, A]` | `ZIO[R, Nothing, A]`     |

## 创建作用

| 函数名                    | 输入                                                                                     | 输出                                | 含义                                                  |
|------------------------|----------------------------------------------------------------------------------------|-----------------------------------|-----------------------------------------------------|
| ZIO.succeed            | `=> A`                                                                                 | `IO[Nothing, A]`                  | 创建成功的返回 A 类型值的作用                                    |
| ZIO.fail               | `=> E`                                                                                 | `IO[E, Nothing]`                  | 创建因为指定错误 E 而失败的作用                                   |
| ZIO.interrupt          |                                                                                        | `IO[Nothing, Nothing]`            | 创建一个视为被打断的作用                                        |
| ZIO.die                | `Throwable`                                                                            | `IO[Nothing, Nothing]`            | 程序以该错误失败，视为 defect，可以打断协程                           |
| ZIO.attempt            | `=> A`                                                                                 | `IO[Throwable, A]`                | 执行代码块，捕获代码块的异常，并在成功时返回对应值，<br />一般用于将外部作用包装为 ZIO 作用 |
| ZIO.async              | `((IO[E, A] => Unit) => Unit)` <br> FiberId                                            | `IO[E, A]`                        | 将异步回调式的 API 转换为 ZIO 作用                              |
| ZIO.fromEither         | `Either[E, A]`                                                                         | `IO[E, A]`                        | 将 Either[E, A] 转换为 ZIO 作用                           |
| ZIO.left               | `A`                                                                                    | `IO[Nothing, Either[A, Nothing]]` | 上面的左值变体                                             |
| ZIO.right              | `B`                                                                                    | `IO[Nothing, Either[Nothing, B]]` | 上面的右值变体                                             |
| ZIO.fromFiber          | `Fiber[E, A]`                                                                          | `IO[E, A]`                        | 创建一个返回传入协程结果的作用                                     |
| ZIO.fromFuture         | `ExecutionContext => Future[A]`                                                        | `IO[Throwable, A]`                | 创建一个返回传入的 Future 结果的作用                              |
| ZIO.fromOption         | `Option[A]`                                                                            | `IO[Option[Nothing], A]`          | 将 Option[A] 转换为 ZIO 作用                              |
| ZIO.none               |                                                                                        | `IO[Nothing, Option[Nothing]]`    | 上面的无值变体                                             |
| ZIO.some               | `A`                                                                                    | `IO[Nothing, Option[A]]`          | 上面的有值变体                                             |
| ZIO.fromTry            | `Try[A]`                                                                               | `IO[Throwable, A]`                | 将 Try[A] 转换为 ZIO 作用                                 |
| ZIO.acquireReleaseWith | `IO[E, A]` (acquire) <br> `A => IO[Nothing, Any]` (release) <br> `A => IO[E, B]` (use) | `IO[E, B]`                        | 将可以释放的资源包装到 ZIO 中，需要同时提供如何创建，使用和释放                  |
| ZIO.when               | `Boolean` <br> `IO[E, A]`                                                              | `IO[E, Option[A]]`                | 当表达式的结果为真时，计算传入的作用，否则直接返回 none                      |
| ZIO.whenZIO            | `IO[E, Boolean]` <br> `IO[E, A]`                                                       | `IO[E, Option[A]]`                | 当作用的结果为真时，计算传入的作用，否则直接返回 none                       |
| ZIO.whenCase           | `A` <br> `PartialFunction[A, IO[E, B]]`                                                | `IO[E, Option[B]]`                | 对传入值 A 进行模式匹配，并根据情况生成新作用                            |
| ZIO.whenCaseZIO        | `IO[E, A]` <br> `PartialFunction[A, IO[E, B]]`                                         | `IO[E, Option[B]]`                | 对传入的作用的结果进行模式匹配，并根据情况生成新作用                          |
| ZIO.filter             | `Iterable[A]` <br> `A => IO[E, Boolean]`                                               | `IO[E, List[A]]`                  | 遍历传入列表，为每个元素计算一次作用结果，过滤所有结果为真的元素并包装为新的作用            |
| ZIO.cond               | `Boolean` <br> `A` <br> `E`                                                            | `IO[E, A]`                        | 当表达式为真时，创建作用直接返回结果，否则创建作用直接返回错误                     |

## 作用转换

| 函数名                | 从                     | 参数                                     | 转换到                      | 含义                                     |
|--------------------|-----------------------|----------------------------------------|--------------------------|----------------------------------------|
| map                | `IO[E, A]`            | `A => B`                               | `IO[E, B]`               | 转换作用内部的值                               |
| mapAttempt         | `IO[Throwable, A]`    | `A => B`                               | `IO[Throwable, B]`       | 转换作用内部的值，并将转换过程中产生的异常包装到 ZIO 的错误中      |
| as                 | `IO[E, A]`            | `B`                                    | `IO[E, B]`               | 将结果替换为 B                               |
| asSome             | `IO[E, A]`            |                                        | `IO[E, Option[A]]`       | 将结果替换为 Some[A]                         |
| asSomeError        | `IO[E, A]`            |                                        | `IO[Option[E], A]`       | 将结果中的错误替换为 Some[E]                     |
| orElseFail         | `IO[E, A]`            | `E2`                                   | `IO[E2, A]`              | 如果错误发生，直接替换为 E2                        |
| unit               | `IO[E, A]`            |                                        | `IO[E, Unit]`            | 将结果直接替换为 Unit                          |
| flatMap            | `IO[E, A]`            | `A => IO[E1, B]`                       | `IO[E1, B]`              | 为当前作用增加下一步                             |
| flatten            | `IO[E, IO[E1, A]]`    |                                        | `IO[E1, A]`              | 将 ZIO 套 ZIO 按照嵌套顺序执行外层内层各一次，并保留内层的结果   |
| mapBoth            | `IO[E, A]`            | `E => E2` <br> `A => B`                | `IO[E2, B]`              | 错误和结果都转换，无论结果是哪个                       |
| mapError           | `IO[E, A]`            | `E => E2`                              | `IO[E2, A]`              | 将结果中的错误转换为另一个错误                        |
| mapErrorCause      | `IO[E, A]`            | `Cause[E] => Cause[E2]`                | `IO[E2, A]`              | 转换错误，并保留错误的原因（cause by）                |
| flatMapError       | `IO[E, A]`            | `E => IO[Nothing, E2]`                 | `IO[E2, A]`              | 转换错误为一个能够产生新错误的作用，并执行他来替换原错误           |
| collect            | `IO[E, A]`            | `E1` <br> `PartialFunction[A, B]`      | `IO[E1, B]`              | 对结果进行模式匹配，并返回转换后的结果；当值不满足任何模式时，返回错误 E2 |
| sandbox            | `IO[E, A]`            |                                        | `IO[Cause[E], A]`        | 展开当前错误的原因                              |
| flip               | `IO[E, A]`            |                                        | `IO[A, E]`               | 将错误和返回值翻转                              |
| tap                | `IO[E, A]`            | `A => IO[E1, _]`                       | `IO[E1, A]`              | 在中途插入作用，并无视结果                          |
| tapBoth            | `IO[E, A]`            | `E => IO[E1, _]` <br> `A => IO[E1, _]` | `IO[E1, A]`              | 在错误端和正确端都插入作用，并无视结果                    |
| tapError           | `IO[E, A]`            | `E => IO[E1, _]`                       | `IO[E1, A]`              | 在错误端插入作用，并无视结果                         |
| absolve            | `IO[E, Either[E, A]]` |                                        | `IO[E, A]`               | 将左值类型与作用的错误类型相同的 Either  返回值展开         |
| some               | `IO[E, Option[A]]`    |                                        | `IO[Option[E], A]`       | 提取 Option 中的值，并给错误套上 Option            |
| head               | `IO[E, List[A]]`      |                                        | `IO[Option[E], A]`       | 提取 List 中第一个值，并给错误套上 Option            |
| toFuture           | `IO[Throwable, A]`    |                                        | `IO[Nothing, Future[A]]` | 转换作用为 Future，并把错误也扔到 Future 中          |
| filterOrDie        | `IO[E, A]`            | `A => Boolean` <br> `Throwable`        | `IO[E, A]`               | 如果断言失败，作用直接以传入的错误失败                    |
| filterOrDieMessage | `IO[E, A]`            | `A => Boolean` <br> `String`           | `IO[E, A]`               | 同上，以 RuntimeException + 传入信息的形式失败      |
| filterOrElse       | `IO[E, A]`            | `A => Boolean` <br> `A => IO[E, A]`    | `IO[E, A]`               | 如果断言失败，生成并返回另一个作用                      |
| filterOrFail       | `IO[E, A]`            | `A => Boolean` <br> `E`                | `IO[E, A]`               | 如果断言失败，返回指定的错误                         |
| when               | `IO[E, A]`            | `Boolean`                              | `IO[E, Option[A]]`       | 当表达式为真时保留值，否则返回 None                   |
| unless             | `IO[E, A]`            | `Boolean`                              | `IO[E, Option[A]]`       | 当表达式为假时保留值，否则返回 None                   |

## 声明并提供环境

| 函数名                | 输入             | 参数                 | 输出                   | 含义 |
|--------------------|----------------|--------------------|----------------------|----|
| ZIO.service        |                | `given Tag[A]`     | `ZIO[A, Nothing, A]` |    |
| provideEnvironment | `ZIO[R, E, A]` | `ZEnvironment[R]`  | `IO[E, A]`           |    |
| provideLayer       | `ZIO[R, E, A]` | `ZLayer[R0, E, R]` | `ZIO[R0, E, A]`      |    |

## Configuration

| Name       | From | Given       | To                    |
|------------|------|-------------|-----------------------|
| ZIO.config |      | `Config[A]` | `IO[Config.Error, A]` |

## Recover from errors

| Name          | From       | Given                                         | To                          |
|---------------|------------|-----------------------------------------------|-----------------------------|
| either        | `IO[E, A]` |                                               | `IO[Nothing, Either[E, A]]` |
| option        | `IO[E, A]` |                                               | `IO[Nothing, Option[A]]`    |
| ignore        | `IO[E, A]` |                                               | `IO[Nothing, Unit]`         |
| exit          | `IO[E, A]` |                                               | `IO[Nothing, Exit[E, A]]`   |
| `<>` (orElse) | `IO[E, A]` | `IO[E2, A1]`                                  | `IO[E2, A1]`                |
| orElseEither  | `IO[E, A]` | `IO[E2, B]`                                   | `IO[E2, Either[A, B]]`      |
| fold          | `IO[E, A]` | `E => B` <br> `A => B`                        | `IO[Nothing, B]`            |
| foldZIO       | `IO[E, A]` | `E => IO[E2, B]` <br> `A => IO[E2, B]`        | `IO[E2, B]`                 |
| foldCauseZIO  | `IO[E, A]` | `Cause[E] => IO[E2, B]` <br> `A => IO[E2, B]` | `IO[E2, B]`                 |
| catchAll      | `IO[E, A]` | `E => IO[E2, A1]`                             | `IO[E2, A1]`                |
| catchAllCause | `IO[E, A]` | `Cause[E] => IO[E2, A1]`                      | `IO[E2, A1]`                |
| catchSome     | `IO[E, A]` | `PartialFunction[E, IO[E1, A1]]`              | `IO[E1, A1]`                |
| retry         | `IO[E, A]` | `Schedule[E, S]`                              | `IO[E, A]`                  |
| eventually    | `IO[E, A]` |                                               | `IO[Nothing, A]`            |

## Terminate fiber with errors

| Name            | From               | Given                                          | To               |
|-----------------|--------------------|------------------------------------------------|------------------|
| orDie           | `IO[Throwable, A]` |                                                | `IO[Nothing, A]` |
| orDieWith       | `IO[E, A]`         | `E => Throwable`                               | `IO[Nothing, A]` |
| refineOrDie     | `IO[Throwable, A]` | `PartialFunction[Throwable, E2]`               | `IO[E2, A]`      |
| refineOrDieWith | `IO[E, A]`         | `PartialFunction[E, E2]` <br> `E => Throwable` | `IO[E2, A]`      |

## Combining effects + parallelism

| Name               | From       | Given                                            | To                               |
|--------------------|------------|--------------------------------------------------|----------------------------------|
| ZIO.foldLeft       |            | `Iterable[A]` <br> `S` <br> `(S, A) => IO[E, S]` | `IO[E, S]`                       |
| ZIO.foreach        |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 |
| ZIO.foreachPar     |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 |
| ZIO.collectAll     |            | `Iterable[IO[E, A]]`                             | `IO[E, List[A]]`                 |
| ZIO.collectAllPar  |            | `Iterable[IO[E, A]]`                             | `IO[E, List[A]]`                 |
| ZIO.forkAll        |            | `Iterable[IO[E, A]]`                             | `IO[Nothing, Fiber[E, List[A]]]` |
| fork               | `IO[E, A]` |                                                  | `IO[Nothing, Runtime[E, A]]`     |
| `<*>` (zip)        | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 |
| `*>` (zipRight)    | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      |
| `<*` (zipLeft)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      |
| `<&>` (zipPar)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 |
| `&>` (zipParRight) | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      |
| `<&` (zipParLeft)  | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      |
| race               | `IO[E, A]` | `IO[E1, A1]`                                     | `IO[E1, A1]`                     |
| raceAll            | `IO[E, A]` | `Iterable[IO[E1, A1]]`                           | `IO[E1, A1]`                     |
| raceEither         | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, Either[A, B]]`           |

## Finalizers

| Name          | From       | Given                      | To         |
|---------------|------------|----------------------------|------------|
| ensuring      | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` |
| onError       | `IO[E, A]` | `Cause[E] => UIO[_]`       | `IO[E, A]` |
| onInterrupt   | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` |
| onTermination | `IO[E, A]` | `Cause[Nothing] => UIO[_]` | `IO[E, A]` |

## Timing and repetition

| Name        | From       | Given            | To                     |
|-------------|------------|------------------|------------------------|
| ZIO.never   |            |                  | `IO[Nothing, Nothing]` |
| ZIO.sleep   |            | `Duration`       | `IO[Nothing, Unit]`    |
| delay       | `IO[E, A]` | `Duration`       | `IO[E, A]`             |
| timeout     | `IO[E, A]` | `Duration`       | `IO[E, Option[A]]`     |
| timed       | `IO[E, A]` |                  | `IO[E, (Duration, A)]` |
| forever     | `IO[E, A]` |                  | `IO[E, Nothing]`       |
| repeat      | `IO[E, A]` | `Schedule[A, B]` | `IO[E, B]`             |
| repeatUntil | `IO[E, A]` | `A => Boolean`   | `IO[E, A]`             |
| repeatWhile | `IO[E, A]` | `A => Boolean`   | `IO[E, A]`             |

## Logging

| Name           | From | Given    | To                  |
|----------------|------|----------|---------------------|
| ZIO.log        |      | `String` | `IO[Nothing, Unit]` |
| ZIO.logFatal   |      | `String` | `IO[Nothing, Unit]` |
| ZIO.logError   |      | `String` | `IO[Nothing, Unit]` |
| ZIO.logWarning |      | `String` | `IO[Nothing, Unit]` |
| ZIO.logInfo    |      | `String` | `IO[Nothing, Unit]` |
| ZIO.logDebug   |      | `String` | `IO[Nothing, Unit]` |
| ZIO.logTrace   |      | `String` | `IO[Nothing, Unit]` |
