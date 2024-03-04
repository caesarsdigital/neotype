# neotype

[![Release Artifacts][Badge-SonatypeReleases]][Link-SonatypeReleases]
[![Snapshot Artifacts][Badge-SonatypeSnapshots]][Link-SonatypeSnapshots]

[Badge-SonatypeReleases]: https://img.shields.io/nexus/r/https/oss.sonatype.org/io.github.kitlangton/neotype_3.svg "Sonatype Releases"
[Badge-SonatypeSnapshots]: https://img.shields.io/nexus/s/https/oss.sonatype.org/io.github.kitlangton/neotype_3.svg "Sonatype Snapshots"
[Link-SonatypeSnapshots]: https://oss.sonatype.org/content/repositories/snapshots/io/github/kitlangton/neotype_3/ "Sonatype Snapshots"
[Link-SonatypeReleases]: https://oss.sonatype.org/content/repositories/releases/io/github/kitlangton/neotype_3/ "Sonatype Releases"

A friendly refined/newtype library for Scala 3.

```scala
"io.github.kitlangton" %% "neotype" % "0.2.0"
```

## Features

- Compile-time Checked Values
- Write validations as **plain, old Scala expressions**
- Helpful compliation errors (_see below_)
- No runtime allocations (Thanks to `inline` and `opaque type`)
- Integrates with other libraries (e.g. `zio-json`, `circe`, `tapir`, etc.)

### Example

Here is how to define a compile-time validated Newtype.

```scala
import neotype.*

// 1. Define a newtype.
object NonEmptyString extends Newtype[String]:

  // 2. Optionally, define a validate method.
  override inline def validate(input: String): Boolean =
    input.nonEmpty

// 3. Construct values.
NonEmptyString("Hello") // OK
NonEmptyString("")      // Compile Error
```

Attempting to call `NonEmptyString("")` would result in the following compilation error:

```scala
Error: /src/main/scala/examples/Main.scala:9:16
  NonEmptyString("")
  ^^^^^^^^^^^^^^^^^^
  —— Newtype Error ——————————————————————————————————————————————————————————
  NonEmptyString was called with an INVALID String.
  input: ""
  check: input.nonEmpty
  ———————————————————————————————————————————————————————————————————————————
```

## Integrations

Neotype integrates with the following libraries.

- zio-test `DeriveGen`
- zio-json
- zio-config
- tapir
- quill
- circe
- jsoniter

### ZIO Json Example

```scala
import neotype.*
import neotype.ziojson.given
import zio.json.*

type NonEmptyString = NonEmptyString.Type
object NonEmptyString extends Newtype[String]:
  override inline def validate(value: String): Result =
    if value.nonEmpty then true else "String must not be empty"

case class Person(name: NonEmptyString, age: Int) derives JsonCodec

val parsed = """{"name": "Kit", "age": 30}""".fromJson[Person]
// Right(Person(NonEmptyString("Kit"), 30))

val failed = """{"name": "", "age": 30}""".fromJson[Person]
// Left(".name(String must not be empty)")
```

By importing `neotype.ziojson.given`, we automatically generate a `JsonCodec` for `NonEmptyString`. Custom
failure messages are also supported (by overriding `def failureMessage` in the Newtype definition).
