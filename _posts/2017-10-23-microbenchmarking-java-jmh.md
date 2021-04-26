---
layout: post
title: Microbenchmarking in Java with JMH
description: Full example of microbenchmarking in Java using JMH
date:  2017-10-23
time: 20 mins
tags: java performance
language: english
---

Although there are great third-party tools out there to measure the performance of our java applications, it's always handy to know options with close-to-zero setup time.

JMH stands for _Java Microbenchmark Harness_ and has been part of the [openjdk core](https://github.com/openjdk/jmh) since 2012.

Using their words:

> JMH is a Java harness for building, running, and analysing nano/micro/milli/macro benchmarks written in Java and other languages targeting the JVM.

Benchmarking key methods and comparing results while your project grows may be a priceless source of information for your team. Going beyond this concept, benchmarking can also be a great tool at development time. Let's see why.

<!-- more -->

## Our base java project to play with

This will be the structure of our project:

```
jmh-project
├──jmh-library
│  ├──pom.xml
│  └──src
│     └──main
│        └──java
└──pom.xml
```

With the following [parent](/assets/microbenchmarking-java-jmh/parent-pom.xml) and [library](/assets/microbenchmarking-java-jmh/library-pom.xml) `pom.xml` files.


## Including some code in our library

For the sake of ease, let's imagine we want to create a service that receives a document's content and counts its words. Let's also try to understand the general idea, so you can comprehend how this simple example can apply to a myriad of similar circumstances you may have.

We'll create a class `DocumentWordsCounter.java` within our library, implementing the task we were asked for.

```java
class DocumentWordsCounter {

	public Map<String, Long> countWords(final String documentContent) {
		final Map<String, Long> wordCounts = new HashMap<>();

		for (final String word : documentContent.split("\\W+")) {
			final Long existingCount = wordCounts.get(word);

			if (existingCount == null) {
				wordCounts.put(word, 1L);
			} else {
				wordCounts.put(word, existingCount + 1);
			}
		}

		return wordCounts;
	}

}
```

At this point, a few other approaches may come to our mind, like replacing the `if` / `else` with a Map [merge](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html#merge-K-V-java.util.function.BiFunction-), or using a Stream [Collector](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html) to [group](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#groupingBy-java.util.function.Function-java.util.stream.Collector-) and [count](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#counting--) the results.

So why not compare the three solutions if we don't know which one would perform better?

```java
Map<String, Long> countWordsClassic(final String documentContent) {
	final Map<String, Long> wordCounts = new HashMap<>();

	for (final String word : documentContent.split("\\W+")) {
		final Long existingCount = wordCounts.get(word);

		if (existingCount == null) {
			wordCounts.put(word, 1L);
		} else {
			wordCounts.put(word, existingCount + 1);
		}
	}

	return wordCounts;
}

Map<String, Long> countWordsMerging(final String documentContent) {
	final Map<String, Long> wordCounts = new HashMap<>();

	for (final String word : documentContent.split("\\W+")) {
		wordCounts.merge(word, 1L, Long::sum);
	}

	return wordCounts;
}

Map<String, Long> countWordsCollecting(final String documentContent) {
	return Arrays.stream(documentContent.split("\\W+"))
			.collect(Collectors.groupingBy(Function.identity(), HashMap::new, counting()));
}
```

Here is where benchmarking tools come to help.

## Setting up the benchmark module

A separate module for benchmarking is recommended to isolate concepts within our project, so let's include the following structure:

```
jmh-project
├──jmh-benchmark
│  ├──pom.xml
│  └──src
│     └──main
│        └──java
└──...
```

Having the new module's `pom.xml` looking [like this](/assets/microbenchmarking-java-jmh/benchmark-pom.xml).

As you can see, we just need to add our library and the JMH dependencies.

```xml
<dependencies>
	<!-- Our library so we can reference it -->
	<dependency>
		<groupId>${project.parent.groupId}</groupId>
		<artifactId>jmh-library</artifactId>
		<version>${project.parent.version}</version>
	</dependency>

	<!-- JMH dependencies -->
	<dependency>
		<groupId>org.openjdk.jmh</groupId>
		<artifactId>jmh-core</artifactId>
		<version>0.5</version>
	</dependency>
	<dependency>
		<groupId>org.openjdk.jmh</groupId>
		<artifactId>jmh-generator-annprocess</artifactId>
		<version>0.5</version>
	</dependency>
</dependencies>
```

Don't forget to include the new module in the parent pom!

```xml
<modules>
	<module>jmh-library</module>
	<module>jmh-benchmark</module>
</modules>
```

## Creating the benchmark class

Our benchmark class will initially include a reference to the service we want to evaluate, and the required annotations to be handled by JMH.

```java
@State(Scope.Benchmark)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class WordCount {

	private DocumentWordsCounter service = new DocumentWordsCounter();

	private String documentContent;

	@Setup(Level.Invocation)
	public void setUp() {
		documentContent = ResourcesUtils.generateDocument(10_000);
	}

	@GenerateMicroBenchmark
	@Warmup(iterations = 2)
	@Measurement(iterations = 10)
	@Fork(value = 1)
	public void countWordsClassic() {
		service.countWordsClassic(documentContent);
	}

}
```

Even though more complete information can be found in the official documentation, here is a short description of the annotations we used.

`@State` - Marks the scope of an object, that can be reused across multiple calls. Although not needed in very basic examples, it gets necessary at class level if we want to use other annotations like `@Setup`. JMH provides three different scopes:

- `Benchmark` - The object is shared for all threads running the benchmark.
- `Group` - Each thread group running the benchmark will create its own instance of the state object.
- `Thread` - Each thread running the benchmark will create its own instance of the state object.

`@BenchmarkMode` - Indicates to JMH the information we want to get. There are five different modes:

- `Throughput`- Number of times per second your method could be executed.
- `AverageTime`- Average time for your benchmark method to execute.
- `SampleTime`- Time to execute your benchmark method, including max and min information.
- `SingleShotTime`- Measures how long a single execution of your benchmark method takes (without warmup).
- `All`- A combination of the above.

`@OutputTimeUnit` - Time unit to be used in the benchmarks, from `java.util.concurrent.TimeUnit`.

`@Setup` - Tells JMH to call this method to set up the state object before it is passed to the benchmark method. There is also a `@Teardown` annotation we can configure to be executed out of the benchmark method but still part of the whole iteration.

`@GenerateMicroBenchmark` - Marks a method to be benchmarked.

`@Warmup` - Number of dry run iterations before JMH starts collecting results (default 20).

`@Measurement` - Number of executions to measure (default 20).

`@Fork` - Number of full repetitions, including warmups (default 10).

So applying them to the above code:
1. We set up the benchmark, telling JMH to collect the average execution, measuring the elapsed time in milliseconds.
2. The method `countWordsClassic` is:
  - annotated as a benchmark method.
  - configured to be executed twice for warmup with `@Warmup(iterations = 2)`.
  - configured to be executed 10 times for measuring with `@Warmup(Measurement = 2)`.
  - configured to has a single full execution with `@Fork(value = 1)`.
3. A `@Setup` method is configured to prepare a document for our benchmark method, from wherever source. It's important for this code to be out of the benchmark method to avoid affecting its actual performance. A basic implementation could be a utility class, generating and caching random documents on-the-fly with a concrete number of words. For example:

```java
public final class ResourcesUtils {

	private static final Map<Integer, String> DOCUMENTS = new HashMap<>();

	public static String generateDocument(final int numWords) {
		if (!DOCUMENTS.containsKey(numWords)) {
			DOCUMENTS.put(numWords, generateRandomDocument(numWords));
		}

		return DOCUMENTS.get(numWords);
	}

	private static String generateRandomDocument(final int numWords) {
		final Random rand = new Random();

		return IntStream.generate(rand::nextInt)
				.limit(numWords)
				.mapToObj(String::valueOf)
				.collect(Collectors.joining(" "));
	}

}
```

### Including iteration parameters

Another cool feature that JMH provides out-of-the-box is the ability to include parameters to be used during the benchmark executions. If we wanted to benchmark how `countWordsClassic` performs with documents of different sizes, we could do it easily with the `@Param` annotation.

```java
@Param({ "100", "1000", "10000", "100000", "1000000" })
private int words;
```

Meaning that we want to repeat the benchmark methods under this class once per value. The field `words` gets automatically populated by JMH with the values included in the `@Param` annotation.

This is a perfect match for our `@Setup` method, because we can use the `words` field to get a dynamic document size.

```java
@Setup(Level.Invocation)
public void setUp() {
	documentContent = ResourcesUtils.generateDocument(words);
}
```

## Executing the benchmarks

There are two chief ways to execute the benchmark methods.

### Executing benchmark methods directly

This is the fastest way to execute benchmark methods from an IDE. We just need to include a `main` method somewhere and invoke JMH from there.

```java
public static void main(String[] args) throws Exception {
	org.openjdk.jmh.Main.main(args);
}
```

On the plus side, this approach will provide quick feedback without having to package the project. However, this solution could hardly be useful out of the development environment. In addition, controlling the local resources for the execution of the benchmarks (memory, CPU, etc) may be a bit pesky.

### Generating a benchmark executable artifact

The other way to execute benchmarks would be to generate a fat jar that may be executed anywhere. This approach is quite useful for CI/CD, and can be easily configured with the [maven shade plugin](https://maven.apache.org/plugins/maven-shade-plugin/).

We'd just need to include the following build configuration to the benchmark module's `pom.xml` file:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-shade-plugin</artifactId>
			<version>2.0</version>
			<executions>
				<execution>
					<phase>package</phase>
					<goals>
						<goal>shade</goal>
					</goals>
					<configuration>
						<finalName>microbenchmarks</finalName>
						<transformers>
							<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
								<mainClass>org.openjdk.jmh.Main</mainClass>
							</transformer>
						</transformers>
						<filters>
							<filter>
								<artifact>*:*</artifact>
								<excludes>
									<exclude>META-INF/services/javax.annotation.processing.Processor</exclude>
								</excludes>
							</filter>
						</filters>
					</configuration>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

This configuration tells maven to execute the shade plugin during the `package` phase, generating an artifact called _microbenchmarks.jar_ (see the `finalName` entry).

So we just need to execute `mvn clean package` to get both our library and benchmark modules packaged, and an extra executable artifact that can be used anytime and anywhere. For sure this plugin could be configured at your convenience, with a custom output name, running under specific profiles, etc.

The new artifact is barely a fat jar, so we can execute it with:

```sh
java -jar microbenchmarks.jar
```

### Analyzing the results

When the benchmark is triggered (using any of the above methods), we'll see information about the progress in the output. The total execution time will depend on the number of warmups, measurements, and forks configured for each benchmark method.

Once finished, we get a final table-like section with the results:

```
Benchmark                     (words)   Mode   Samples         Mean   Mean error    Units
WordCount.countWordsClassic       100   avgt        10        0.012        0.000    ms/op
WordCount.countWordsClassic      1000   avgt        10        0.146        0.005    ms/op
WordCount.countWordsClassic     10000   avgt        10        1.524        0.040    ms/op
WordCount.countWordsClassic    100000   avgt        10       22.220        2.526    ms/op
WordCount.countWordsClassic   1000000   avgt        10      351.001       74.342    ms/op
```

This summary includes the results for each iteration (based on the `@Param` values), including the average execution time (`Mean` column) and the average deviation (`Mean error`).

## Benchmarking our three approaches

It's now trivial to benchmark the other two approaches `countWordsMerging` and `countWordsCollecting`,
getting an insightful idea for our final decision.

```java
@GenerateMicroBenchmark
@Warmup(iterations = 2)
@Measurement(iterations = 10)
@Fork(value = 1)
public void classic() {
	service.countWordsClassic(documentContent);
}

@GenerateMicroBenchmark
@Warmup(iterations = 2)
@Measurement(iterations = 10)
@Fork(value = 1)
public void merging() {
	service.countWordsMerging(documentContent);
}

@GenerateMicroBenchmark
@Warmup(iterations = 2)
@Measurement(iterations = 10)
@Fork(value = 1)
public void collecting() {
	service.countWordsCollecting(documentContent);
}
```

After a while...

```
Benchmark              (words)   Mode   Samples         Mean   Mean error    Units
WordCount.classic          100   avgt        10        0.013        0.001    ms/op
WordCount.classic         1000   avgt        10        0.145        0.004    ms/op
WordCount.classic        10000   avgt        10        1.493        0.043    ms/op
WordCount.classic       100000   avgt        10       25.992        4.608    ms/op
WordCount.classic      1000000   avgt        10      330.102       26.725    ms/op
WordCount.collecting       100   avgt        10        0.014        0.000    ms/op
WordCount.collecting      1000   avgt        10        0.163        0.010    ms/op
WordCount.collecting     10000   avgt        10        1.739        0.069    ms/op
WordCount.collecting    100000   avgt        10       28.537        9.564    ms/op
WordCount.collecting   1000000   avgt        10      378.990       18.523    ms/op
WordCount.merging          100   avgt        10        0.013        0.001    ms/op
WordCount.merging         1000   avgt        10        0.143        0.008    ms/op
WordCount.merging        10000   avgt        10        2.844        2.006    ms/op
WordCount.merging       100000   avgt        10       21.430        4.760    ms/op
WordCount.merging      1000000   avgt        10      257.388       19.870    ms/op
```

We can see that, although the three approaches are quite similar in terms of performance, `countWordsMerging` looks slightly better.
