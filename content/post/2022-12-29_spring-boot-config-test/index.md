---
title: "Spring Boot でプロファイルに応じて読み込む設定ファイルの設定値をテストする"
author: ["yewton"]
date: 2022-12-29T23:53:00+09:00
mylastmod: 2022-12-30T00:06:35+09:00
slug: "spring-boot-config-test"
tags: ["spring", "spring-boot", "kotest", "Java", "Kotlin"]
categories: ["技術系"]
draft: false
---

## 参考 {#参考}

-   [Testing Spring Boot @ConfigurationProperties | Baeldung](https://www.baeldung.com/spring-boot-testing-configurationproperties)
-   [【2021】SpringBootでpropertiesやymlの設定ファイルが読み込めることのテストを書く - きり丸の技術日記](https://nainaistar.hatenablog.com/entry/2021/03/30/120000)


## 本文 {#本文}

[公式リファレンスドキュメント](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.external-config) に記載されているように、 Spring Boot での外部設定ファイル読み込み方法は色々ある。

`application.(yml|properties)` と、 [Profile](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.profiles) 毎に `application-production.(yml|properties)` などを配置して [プロファイル固有の設定を記述](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.external-config.files.profile-specific) するのを基本として、
[`spring.config.import`](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.external-config.files.importing) で追加の設定ファイルを読み込んだり、
[YAML のマルチドキュメント](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.external-config.files.multi-document) と `spring.config.activate.on-profile` などを使って [特定の条件で Property を有効化](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.external-config.files.activation-properties) したり、
[Profile Groups](https://docs.spring.io/spring-boot/docs/3.0.1/reference/htmlsingle/#features.profiles.groups) でプロファイルをグループ化したり、等々。

色々なやり方がある故に、例えば本番環境用など特定のプロファイルでベースの設定をオーバーライドしたい、というような場合に、ちょっと手の込んだ記述をしていたりすると、本当に意図した通りに値が設定されているか不安になることがある。

実際にそのプロファイルで動かしてみるまで分からないのでは困るので、 test で確認出来るようにしてみる。

実際のテストコードはこちら( [ProductionConfigTest.kt](https://github.com/yewton/asobiba/blob/8af5b30f3c66669b1d83988713b98e6d32ef6aff/reactive-webapp/src/test/kotlin/net/yewton/asobiba/reactivewebapp/ProductionConfigTest.kt) )。
[Kotest](https://kotest.io/) での記述になっているが、 Kotest 固有の部分は無い。

全容は以下の通り。

```kotlin
@ContextConfiguration(initializers = [ConfigDataApplicationContextInitializer::class])
@ActiveProfiles("production")
class ProductionConfigTest(env: ConfigurableEnvironment) : FunSpec({
    test("R2DBC設定") {
        env.getProperty("spring.r2dbc.url") shouldContain "prod.yewton.net:5432"
    }
    test("Redis設定") {
        env.getProperty("spring.data.redis.host") shouldContain "prod.yewton.net"
    }
})
```

```kotlin
@ContextConfiguration(initializers = [ConfigDataApplicationContextInitializer::class])
```

[`@ContextConfiguration`](https://docs.spring.io/spring-framework/docs/6.0.3/javadoc-api/org/springframework/test/context/ContextConfiguration.html) に [`org.springframework.boot.test.context.ConfigDataApplicationContextInitializer`](https://docs.spring.io/spring-boot/docs/3.0.1/api/org/springframework/boot/test/context/ConfigDataApplicationContextInitializer.html) を指定することで、 `application.properties` 等の設定ファイルをロードするよう指定出来るのは、冒頭の参考リンクにある通り。

```kotlin
@ActiveProfiles("production")
```

[@ActiveProfiles](https://docs.spring.io/spring-framework/docs/6.0.3/javadoc-api/org/springframework/test/context/ActiveProfiles.html) でテストしたいプロファイルを指定する。

```kotlin
class ProductionConfigTest(env: ConfigurableEnvironment) : FunSpec({
```

設定値は [`ConfigurableEnvironment`](https://docs.spring.io/spring-framework/docs/6.0.3/javadoc-api/org/springframework/core/env/ConfigurableEnvironment.html) を autowire して取得する。

{{% callout info %}}
`ConfigurableEnvironment` は、 内部的に複数の [`PropertySource`](https://docs.spring.io/spring-framework/docs/6.0.3/javadoc-api/org/springframework/core/env/PropertySource.html) を保持しており、これらを使ってプロパティの値を解決する。

試しにデバッガで見てみると、読み込まれているファイル群を確認出来る:

{{< figure src="2022-12-29_23-31-26_screenshot.png" >}}

ちなみに各 `PropertySource#source` に、 `Map` として設定値が格納されている:

{{< figure src="2022-12-29_23-48-03_screenshot.png" >}}

これを利用して、設定キーの一覧を作成したりすることも可能。
{{% /callout %}}

```kotlin
env.getProperty("spring.r2dbc.url") shouldContain "prod.yewton.net:5432"
```

[`PropertyResolver#getProperty`](2022-12-29_23-31-26_screenshot.png) メソッドで、特定のプロパティの値を取得出来る。

この値を利用してどういったテストを書くかは、状況に応じて。