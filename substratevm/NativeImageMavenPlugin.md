# Native Image Maven Plugin

To simplify the generation of native images, Native Image now works out
of [Maven](https://maven.apache.org/what-is-maven.html) with the [Native Image Maven Plugin](https://search.maven.org/artifact/com.oracle.substratevm/native-image-maven-plugin/).

One can build a native image directly with Maven
using the `mvn package` command without running the `native-image` tool as a
separate step. It is sufficient to add `native-image-maven-plugin` into the
`<plugins>` section of the `pom.xml` file:
```
<plugin>
    <groupId>org.graalvm.nativeimage</groupId>
    <artifactId>native-image-maven-plugin</artifactId>
    <version>${graalvm.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>native-image</goal>
            </goals>
            <phase>package</phase>
        </execution>
    </executions>
    <configuration>
        <skip>false</skip>
        <imageName>example</imageName>
        <buildArgs>
            --no-fallback
        </buildArgs>
    </configuration>
</plugin>
```
and the `org.graalvm.sdk` library dependency in the `<dependencies>` list:

```
<dependency>
    <groupId>org.graalvm.sdk</groupId>
    <artifactId>graal-sdk</artifactId>
    <version>${graalvm.version}</version>
    <scope>provided</scope>
</dependency>
```

The plugin figures out what jar files it needs to pass to the native image and
what the executable main class should be. If the heuristics fails with the `no
main manifest attribute, in target/<name>.jar` error, the main class should be
specified in the `<configuration>` node of the plugin (see Plugin Customization
section). When `mvn package` completes, an executable, generated in the _target_
directory of the project, is ready for use.

### Maven Plugin Customization

When using GraalVM Enterprise as the `JAVA_HOME` environment, the plugin builds a native image with enterprise features enabled, e.g., an executable will automatically be built with [compressed references](https://medium.com/graalvm/isolates-and-compressed-references-more-flexible-and-efficient-memory-management-for-graalvm-a044cc50b67e) and other optimizations enabled.

It is also possible to customize `native-image-maven-plugin` within a
`<configuration>` node. The following configurations are available.

1. Configuration parameter `<mainClass>`. If the execution fails with the `no main manifest attribute, in target/<name>.jar` error, the main class should be specified. By default the plugin consults several locations in the  `pom.xml` file in the following order to determine what the main class of the image should be:
* `<maven-shade-plugin> <transformers> <transformer> <mainClass>`
* `<maven-assembly-plugin> <archive> <manifest> <mainClass>`
* `<maven-jar-plugin> <archive> <manifest> <mainClass>`
2. Configuration parameter `<imageName>`. If an image filename is not set explicitly, use parameter `<imageName>` to provide a custom filename for the image.
3. Configuration parameter `<buildArgs>`. If you want to pass additional options for the image building, use the `<buildArgs>` parameter the definition of the plugin. For example, to build a native image with assertions enabled that uses _com.test.classname_ as a main class, add:

```
<configuration>
    <imageName>executable-name</imageName>
    <mainClass>com.test.classname</mainClass>
    <buildArgs>
        --no-fallback
    </buildArgs>
    <skip>false</skip>
</configuration>
```
