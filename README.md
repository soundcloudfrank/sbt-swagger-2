# sbt-swagger-2

This plugin is a sbt wrapper for a [Swagger Akka HTTP](https://github.com/swagger-akka-http/swagger-akka-http), that
allows you to generate one (or more) Swagger JSON files during build instead of doing it in runtime.

## Motivation

 * I wanted to limit dependencies used in runtime - as I used Circe and Jaws for JSON, shipping separate library
   (Jackson) as well as quite a lot other dependencies just for Swagger seems like unnnecessary overhead,
 * annotation-based Swagger generators use runtime reflection - for some people that solution itself is a disadvantage,
 * when I tried to create a native image in GraalVM, it failed due to usage of reflection - moving Swagger generation
   to build stage is a part of the effort of overall removal of runtime reflection in my projects,
 * [https://github.com/hootsuite/sbt-swagger](sbt-swagger) is outdated: it isn't migrated to sbt 1.0.0 and uses older
   version of Swagger.

## Usage

Project basically reuse all constructs from [Swagger Akka HTTP](https://github.com/swagger-akka-http/swagger-akka-http).
It add aliases for them in `io.scalaland.sbtswagger2.SbtSwagger2Plugin.autoImport`, so classes used for creating config
can be found within `Swagger` object.

```scala
// API v1
swaggerOutputs += Swagger.Output(
  inputFilter = clazz => Set(
    "backend.healthcheck",
    "backend.auth",
    "backend.api.v1"
  ).exists(prefix => clazz.getName.startsWith(prefix)),
  output = (Compile / classDirectory).value / "docs" / "v1" / "swagger.json",
  host = "http://localhost",
  schemes = List(Swagger.Scheme.HTTP, Swagger.Scheme.HTTPS),
  securitySchemeDefinitions = Map("token" -> new Swagger.OAuth2Definition().password("/auth")),
  info = Swagger.Info(
    title = "Backend API v1",
    version = version.value,
    description = """V1 API description""".stripMargin
  )
)

// API v2
swaggerOutputs += Swagger.Output(
  inputFilter = clazz => Set(
    "backend.healthcheck",
    "backend.auth",
    "backend.api.v2"
  ).exists(prefix => clazz.getName.startsWith(prefix)),
  ...
)
```

It adds however whilelisting ability, so that one could cherry-pick classes to use in each of (potentially) many files.

Once configured one can run it with:

```scala
sbt> swaggerGenerate
```

## Requirements

This plugin requires sbt 1.0.0+. It does *not* add Swagger annotations to `libraryDependencies`.
