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
| ZIO.fromEither         | `Either[E, A]`                                                                         | `IO[E, A]`                        | 将 Either[E, A] 转换为 ZIO 作用                           |
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

| 函数名                | 对象             | 参数                 | 输出                   | 含义          |
|--------------------|----------------|--------------------|----------------------|-------------|
| ZIO.service        |                | `given Tag[A]`     | `ZIO[A, Nothing, A]` | 访问当前环境      |
| provideEnvironment | `ZIO[R, E, A]` | `ZEnvironment[R]`  | `IO[E, A]`           | 将环境传给作用     |
| provideLayer       | `ZIO[R, E, A]` | `ZLayer[R0, E, R]` | `ZIO[R0, E, A]`      | 将带有环境的层传给作用 |

## 配置

| 函数名        | 对象 | 参数          | 输出                    | 含义             |
|------------|----|-------------|-----------------------|----------------|
| ZIO.config |    | `Config[A]` | `IO[Config.Error, A]` | 从指定的配置提供者中读取配置 |

## 从错误中恢复

| 函数名           | 从          | 参数                                            | 转换到                         | 含义                                           |
|---------------|------------|-----------------------------------------------|-----------------------------|----------------------------------------------|
| either        | `IO[E, A]` |                                               | `IO[Nothing, Either[E, A]]` | 将错误和值包装到 Either                              |
| option        | `IO[E, A]` |                                               | `IO[Nothing, Option[A]]`    | 将结果包装为 Some[A]，错误视为 None                     |
| ignore        | `IO[E, A]` |                                               | `IO[Nothing, Unit]`         | 忽略结果和错误，都转换为 Unit                            |
| exit          | `IO[E, A]` |                                               | `IO[Nothing, Exit[E, A]]`   | 将错误转换为程序的退出值                                 |
| `<>` (orElse) | `IO[E, A]` | `IO[E2, A1]`                                  | `IO[E2, A1]`                | 在出错时替换为传入的作用                                 |
| orElseEither  | `IO[E, A]` | `IO[E2, B]`                                   | `IO[E2, Either[A, B]]`      | 在出错时，计算传入的作用，成功则返回 Right[B]，原作用成功则返回 Left[A] |
| fold          | `IO[E, A]` | `E => B` <br> `A => B`                        | `IO[Nothing, B]`            | 将值和错误都转换为统一类型 B                              |
| foldZIO       | `IO[E, A]` | `E => IO[E2, B]` <br> `A => IO[E2, B]`        | `IO[E2, B]`                 | 将值和错误都转换为新作用                                 |
| foldCauseZIO  | `IO[E, A]` | `Cause[E] => IO[E2, B]` <br> `A => IO[E2, B]` | `IO[E2, B]`                 | 将值和错误的原因都转换为新作用                              |
| catchAll      | `IO[E, A]` | `E => IO[E2, A1]`                             | `IO[E2, A1]`                | 捕获错误并转换为新作用                                  |
| catchAllCause | `IO[E, A]` | `Cause[E] => IO[E2, A1]`                      | `IO[E2, A1]`                | 捕获错误原因并转换为新作用                                |
| catchSome     | `IO[E, A]` | `PartialFunction[E, IO[E1, A1]]`              | `IO[E1, A1]`                | 对错误进行模式匹配，从而只处理指定类型的错误                       |
| retry         | `IO[E, A]` | `Schedule[E, S]`                              | `IO[E, A]`                  | 对错误进行重试                                      |
| eventually    | `IO[E, A]` |                                               | `IO[Nothing, A]`            | 忽略错误并重试该作用直到作用成功                             |

## 用错误结束协程

| 函数名             | 对象                 | 参数                                             | 输出               | 含义                                       |
|-----------------|--------------------|------------------------------------------------|------------------|------------------------------------------|
| orDie           | `IO[Throwable, A]` |                                                | `IO[Nothing, A]` | 将作用的错误视为 defect                          |
| orDieWith       | `IO[E, A]`         | `E => Throwable`                               | `IO[Nothing, A]` | 将作用的错误转换为其他异常并视为 defect                  |
| refineOrDie     | `IO[Throwable, A]` | `PartialFunction[Throwable, E2]`               | `IO[E2, A]`      | 将一部分错误值包装为新的错误，并将未处理的错误视为 defect         |
| refineOrDieWith | `IO[E, A]`         | `PartialFunction[E, E2]` <br> `E => Throwable` | `IO[E2, A]`      | 将一部分错误值包装为新的错误，并将未处理的错误转换为其他异常后视为 defect |

## 组合作用 + 并行

| 函数名                | 对象         | 参数                                               | 输出                               | 含义                               |
|--------------------|------------|--------------------------------------------------|----------------------------------|----------------------------------|
| ZIO.foldLeft       |            | `Iterable[A]` <br> `S` <br> `(S, A) => IO[E, S]` | `IO[E, S]`                       | 按集合顺序以每两个作用的结果为参数计算，并最终得到一个作用    |
| ZIO.foreach        |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 | 按集合顺序依次以每个作用的结果为参数计算，并保留其结构      |
| ZIO.foreachPar     |            | `Iterable[A]` <br> `A => IO[E, B]`               | `IO[E, List[B]]`                 | 并行地以集合中每个作用的结果为参数计算，保留结构但不保证顺序   |
| ZIO.collectAll     |            | `Iterable[IO[E, A]]`                             | `IO[E, List[A]]`                 | 依次执行一组作用，并将结果放在原集合类型中返回          |
| ZIO.collectAllPar  |            | `Iterable[IO[E, A]]`                             | `IO[E, List[A]]`                 | 并行执行一组作用，并将结果放在原集合类型中返回，不保证顺序    |
| ZIO.forkAll        |            | `Iterable[IO[E, A]]`                             | `IO[Nothing, Fiber[E, List[A]]]` | 将集合中的每个作用都转换为协程，按原顺序返回转换后的协程对象   |
| fork               | `IO[E, A]` |                                                  | `IO[Nothing, Runtime[E, A]]`     | 将单个作用转换为协程                       |
| `<*>` (zip)        | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 | 将两个作用绑定顺序执行，并返回全部值               |
| `*>` (zipRight)    | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      | 将两个作用绑定顺序执行，丢弃前一个值               |
| `<*` (zipLeft)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      | 将两个作用绑定顺序执行，只保留前一个值              |
| `<&>` (zipPar)     | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, (A, B)]`                 | 同时执行两个作用                         |
| `&>` (zipParRight) | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, B]`                      | 同时执行两个作用，只保留第二个值                 |
| `<&` (zipParLeft)  | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, A]`                      | 同时执行两个作用，只保留第一个值                 |
| race               | `IO[E, A]` | `IO[E1, A1]`                                     | `IO[E1, A1]`                     | 同时执行两个作用，只保留最先返回的值，并立刻打断另一个作用    |
| raceAll            | `IO[E, A]` | `Iterable[IO[E1, A1]]`                           | `IO[E1, A1]`                     | 与另一组作用同时执行，只保留最先返回的值，并立刻打断其他所有作用 |
| raceEither         | `IO[E, A]` | `IO[E1, B]`                                      | `IO[E1, Either[A, B]]`           | 同时执行两个作用，以 Either 的形式保留最先返回的值    |

## 结束操作

| 函数名           | 对象         | 输入                         | 输出         | 含义                                   |
|---------------|------------|----------------------------|------------|--------------------------------------|
| ensuring      | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` | 一旦原作用开始执行，传入的作用就一定会执行，无论原作用成功失败还是被打断 |
| onError       | `IO[E, A]` | `Cause[E] => UIO[_]`       | `IO[E, A]` | 在原作用失败时，生成并执行新作用，新作用不会被打断            |
| onInterrupt   | `IO[E, A]` | `UIO[_]`                   | `IO[E, A]` | 在原作用被打断时，执行新作用                       |
| onTermination | `IO[E, A]` | `Cause[Nothing] => UIO[_]` | `IO[E, A]` | 当原作用因为 defect 或被打断而终结时，生成并执行新作用      |

## 定时和重复

| 函数名         | 对象         | 输入               | 输出                     | 含义                  |
|-------------|------------|------------------|------------------------|---------------------|
| ZIO.never   |            |                  | `IO[Nothing, Nothing]` | 创建一个永远不会结束且什么也不做的作用 |
| ZIO.sleep   |            | `Duration`       | `IO[Nothing, Unit]`    | 创建一个等待一段时间的作用       |
| delay       | `IO[E, A]` | `Duration`       | `IO[E, A]`             | 使作用延迟一段时间后执行        |
| timeout     | `IO[E, A]` | `Duration`       | `IO[E, Option[A]]`     | 给作用增加超时时间           |
| timed       | `IO[E, A]` |                  | `IO[E, (Duration, A)]` | 一并返回该作用的值和执行时间      |
| forever     | `IO[E, A]` |                  | `IO[E, Nothing]`       | 无限循环执行当前作用          |
| repeat      | `IO[E, A]` | `Schedule[A, B]` | `IO[E, B]`             | 按照配置重复执行当前作用        |
| repeatUntil | `IO[E, A]` | `A => Boolean`   | `IO[E, A]`             | 重复执行当前作用，直到作用结果满足断言 |
| repeatWhile | `IO[E, A]` | `A => Boolean`   | `IO[E, A]`             | 当作用结果满足断言时，重复执行当前作用 |

## 日志（字面含义）

| 函数名            | 输入       | 输出                  |
|----------------|----------|---------------------|
| ZIO.log        | `String` | `IO[Nothing, Unit]` |
| ZIO.logFatal   | `String` | `IO[Nothing, Unit]` |
| ZIO.logError   | `String` | `IO[Nothing, Unit]` |
| ZIO.logWarning | `String` | `IO[Nothing, Unit]` |
| ZIO.logInfo    | `String` | `IO[Nothing, Unit]` |
| ZIO.logDebug   | `String` | `IO[Nothing, Unit]` |
| ZIO.logTrace   | `String` | `IO[Nothing, Unit]` |
