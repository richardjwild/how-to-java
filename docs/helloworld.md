## Hello, world!

### Our first class

To begin, create a directory structure to house the project:

```shell
$ mkdir -p helloworld/src/how/to/java
```

Now create a file `helloworld/src/how/to/java` called `HelloWorld.java` with this content:

```java
package how.to.java;

public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

#### Anatomy of a Java class

A Java source file always contains one class definition, ignoring 'inner' classes for the moment. The source file must be named the same as the class with a `.java` suffix, and Java is case sensitive. If the file name does not correspond to the class name, it will not compile.

The first line of the file is the package declaration:

```java
package how.to.java;
```

Java source files must be in a directory structure that exactly mirrors their package declaration. Therefore, one of the purposes served by Java packages is to let you organise your source files into some sort of project structure. A second purpose of Java packages is to provide a namespace for the classes, so we could have another class called `HelloWorld` in another package and they would be different classes. We may unambiguously refer to them by using their *fully qualified* name: `how.to.java.HelloWorld`. The third purpose of Java packages is to give a degree of control over access to classes and their contents.

The next line is a method declaration:

```java
public static void main(String[] args) {
```

The `static` modifier means that the method belongs to the class, not to an instance of a class, therefore you do not need an instance of `HelloWorld` in order to execute the `main` method. You can do it with:

```java
{
    HelloWorld.main(args);
}
```

The signature of this particular method is special, and it makes the `HelloWorld` class a "main" class. That means it can be used as the entry point for an application. A Java application can contain any number of main classes. We will see how to run it shortly.

### Compiler basics

To compile our program, ensure that you are in the `helloworld` directory, and execute this command:

```shell
$ javac src/how/to/java/HelloWorld.java
```

Notice that this has created a new file with a `.class` suffix alongside the original source file:

```shell
$ ls -l src/how/to/java
total 16
-rw-r--r--  1 richardwild  staff  439  3 May 15:35 HelloWorld.class
-rw-r--r--  1 richardwild  staff  147  3 May 15:34 HelloWorld.java
```

To run the program, execute this command:

```shell
$ java -classpath src how.to.java.HelloWorld
Hello, world!
```

#### Classpath

Notice here that we have used the fully qualified name of the `HelloWorld` class to run it. The `-classpath` argument to the JVM tells it where to look for compiled classfiles; in this case we told it to look in the `src` directory. It will thus expect to find the class file for `how.to.java.HelloWorld` at `src/how/to/java/HelloWorld.class` and so it does.

Since it is a path, you can have as many directories on your classpath as you want, and they are separated in the usual operating system-specific way, e.g. by colons on Unix-based systems and by semicolons on Windows.

You can also define an environment variable `CLASSPATH` and then there is no need to pass the `-classpath` argument to the runtime:

```shell
$ export CLASSPATH=src
$ java how.to.java.HelloWorld                                                                                        !10198
Hello, world!
```

#### Destination directory

In practice, we never want the class files to go in the same directory as the java files. We can specify a different directory for the compiled code using the `-d` compiler switch:

```shell
$ mkdir out
$ javac -d out src/how/to/java/HelloWorld.java
$ ls -l out/how/to/java
total 8
-rw-r--r--  1 richardwild  staff  439  3 May 16:23 HelloWorld.class
```

As you can see it has created the directories for the package structure, although the directory specified by the `-d` option must already exist.

Running the program is the same as before, except that now we provide the `out` directory as the classpath:

```shell
$ java -classpath out how.to.java.HelloWorld
Hello, world!
```

#### Sourcepath

So far, we've been compiling a program containing one class, but in the real world applications are composed of many classes. This raises the question of how does the compiler resolve dependencies. Let's answer that by creating one. Add a second source file in `src/how/to/java` called `Greeting.java`:

```java
package how.to.java;

public class Greeting {

    public void greet() {
        System.out.println("Pleased to meet you!");
    }
}
```

Now modify the original `HelloWorld.java` file so that it depends on the new class:

```java
package how.to.java;

public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, world!");
        Greeting greeting = new Greeting();
        greeting.greet();
    }
}
```

Now when we try to compile `HelloWorld.java` we get an error:

```shell
 javac -d out src/how/to/java/HelloWorld.java
src/how/to/java/HelloWorld.java:7: error: cannot find symbol
        Greeting greeting = new Greeting();
        ^
  symbol:   class Greeting
  location: class HelloWorld
src/how/to/java/HelloWorld.java:7: error: cannot find symbol
        Greeting greeting = new Greeting();
                                ^
  symbol:   class Greeting
  location: class HelloWorld
2 errors
```

> Note: If your code actually compiled successfully, it might be because you have your classpath set in the `CLASSPATH` environment variable. Unset the variable and try again.

We can fix this by adding a `-sourcepath` option switch to the compiler:

```shell
$ javac -d out -sourcepath src src/how/to/java/HelloWorld.java
```

Now we see that the compiler has compiled the `Greeting` class even though we did not explicitly tell it to:

```shell
$ ls -l out/how/to/java
total 16
-rw-r--r--  1 richardwild  staff  418  3 May 16:36 Greeting.class
-rw-r--r--  1 richardwild  staff  508  3 May 16:36 HelloWorld.class
```

and the program will run correctly:

```shell
$ java -classpath out how.to.java.HelloWorld
Hello, world!
Pleased to meet you!
```

However, you can pass more than one Java source file to the compiler at once, and when you do this then it will happily resolve the dependencies itself, providing all the required class definitions are there:

```shell
$ javac -d out src/how/to/java/HelloWorld.java \
src/how/to/java/Greeting.java
```

## Conclusion

You will never need to compile Java this way again. The standard tools hide all this stuff from you so you don't need to worry about it. Nevertheless, it is worth understanding how things work at the most basic level, because it may save confusion later that might arise if you don't have this knowledge.