# Firefly

------

Lightweight thrift client code generator and runtime library for Android and Java.

Code generated by firefly has much fewer methods(about 1%) and lines than standard thrift code, which helps applications avoid the notorious 64k limit on methods in Android applications.

For exmaple, evernote's [public api](https://github.com/evernote/evernote-thrift) is provided with thrift. Methods count of standard thrift generated code:

```
<root>: 20815
    com: 20815
        evernote: 20815
            edam: 20815
                error: 226
                limits: 2
                notestore: 15225
                type: 2776
                userstore: 2586
Overall method count: 20815
```

Methods count of Firefly generated code:

```
<root>: 204
    com: 204
        evernote: 204
            edam: 204
                error: 10
                limits: 1
                notestore: 90
                type: 85
                userstore: 18
Overall method count: 204
```

Firefly has an acceptable performance. Local process takes several tens of milliseconds when cold booting a service, which thrift code takes several milliseconds. Hot call takes less than one millisecond almost as fast as thrift code. Anyway, the time taken by local process is ignorable, comparing to the time taken by network.

This project is inspired by Square's [wire](https://github.com/square/wire) and [retrofit](https://github.com/square/retrofit), and I shamelessly copy a lot of code from Twitter's [scrooge](https://github.com/twitter/scrooge) and Google's [gson](https://github.com/google/gson).

## Compiling thrift files
Preparation:

* clone this repository
* install sbt
 
To compile thrift files:

```bash
$ cd generator
$ sbt 'run [--output <output-dir>] <thrift-file1> [<thrift-file2> ...]'
```
If you want to generate files with Rx smell or supports Android's `Parcelable`, just put `--rx` or `--android` in the last, like this:

```bash
$ cd generator
$ sbt 'run [--output <output-dir>] <thrift-file1> [<thrift-file2> ...] --rx --android'
```

If you generate codes with `--android`, your project should dependents on `com.meituan.firefly:lib-android` instead of `com.meituan.firefly:library`.

### Gradle plugin
If you are using Gradle, you may use the Gradle Plugin:

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.meituan.firefly:gradle-plugin:0.2.2'
	}
}
apply plugin: 'com.meituan.firefly'
```
you can change the input diretory and  output diretory, like this:

```groovy
apply plugin: 'com.meituan.firefly'
apply plugin: 'com.android.library'
firefly {
    inputDir file('./src/main/idl')
    outputDir file('./build/generated/source/firefly')
}
```
* inputDir's default value is ./src/main/idl
* outputDir's default value is ./build/generated/source/firefly
* rxStyle's default value is false
* android's default value is false

Please notice that, the plugin must be declared before plugin `com.android.library`, `com.android.application` or `java`.
Pay attention to the arguments `rxStyle` and `android`. If you change them, all the output file will be re-generated.

If you want to generate code with rx smell or support android's Parcelable: 

```groovy
apply plugin: 'com.meituan.firefly'
apply plugin: 'java'
firefly {
    rxStyle true
    android true
}
```

The generator assumes that thrift files included by the target thrift file are placed in the same dir of the target thrift file. 

## Using Firefly in your application
The `library` package contains runtime support libraries that must be included in applications that use Firefly-generated code.

Include via Maven:

```xml
<dependency>
  <groupId>com.meituan.firefly</groupId>
  <artifactId>library</artifactId>
  <version>0.2.2</version>
</dependency>
```

or Gradle:

```groovy
compile 'com.meituan.firefly:library:0.2.2'
```

or sbt:

```scala
libraryDependencies += "com.meituan.firefly" % "library" % "0.2.2"
```

Given the following thrift file for example:

```
namespace java com.meituan.greeting
service GreetingService{
    string greet(string name);
}
```

It generates an interface named `GreetingService`.

Use of generated code:

```Java
final OkHttpClient okhttpClient = new OkHttpClient();
 
GreetingService greetingService = Thrift.instance.create(GreetingService.class, new SimpleProtocolFactory(){
    @Override
    public TProtocol get() {
        return new TBinaryProtocol(new OkhttpTransport(YOUR_SERVICE_URL, okhttpClient));
    }
};

String greeting = greetingService.greet("Han meimei");
```


Make sure that `ProtocolFactory.get()` return a new Protocol for each call, or return a same thread-safe Protocol for every call.

## Roadmap

- [x] RxJava support 
- [x] Gradle Plugin generates codes automatically
- [x] Serializable support
- [x] Android Parcelable support

## License

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

