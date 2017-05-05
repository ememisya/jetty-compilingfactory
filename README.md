# jetty-compilingfactory

Simple jetty module to allow for Java code compilation and execution in Jetty XML configuration.

*Tested with versions jetty 9.3.x - 9.4.x*

## Installation

Simply copy modules and etc contents to your jetty path, and add the following to your start configuration.

```
--module=compilingfactory
```

  This module allows for native Java code to be executed from configuration xml files.
   You may *Call* class edu.vt.middleware.jetty.ext.CompilingClassFactory#loadClass anywhere during configuration
   to compile a Java class.
  
CompilingClassFactory has two *static* methods which may be accessed at anytime.

```java
  public Class<?> loadClass(final String name, final String source) throws IllegalStateException;
  public Class<?> loadClass(final ClassLoader parent, final String name, final String source) throws IllegalStateException;
```
 * @param parent Parent class loader to use after compilation
 * @param name Name of the class including package path (i.e: net.java.ext.MyClass)
 * @param source Source-code of the class to compile


*Example Usage*:

```xml
<!-- declare class -->

<Call id="hideRootContextClass" class="edu.vt.middleware.jetty.ext.CompilingClassFactory" name="loadClass">
          <Arg>HideRootContext</Arg>
          <Arg><![CDATA[
          import java.io.IOException;
          import java.nio.file.DirectoryStream;
          import java.nio.file.FileSystems;
          import java.nio.file.Files;
          import java.nio.file.Path;
        
         /**
          * Class intended to hide the root context before deployment is
          * configured.
          */
          public class HideRootContext
          {
            public void hideRootContext(String contextPath)
            {
            final DirectoryStream<Path> stream;
            try {
              stream = Files.newDirectoryStream(FileSystems.getDefault().getPath(contextPath));
            } catch (IOException iOException) {
              throw new RuntimeException(iOException);
            }
            stream.forEach((java.nio.file.Path t) -> {
              try {
                //Hide root context before deployment
                if ("root".equals(t.toFile().getName()) && t.toFile().isDirectory()) {
                  final Path hiddenFile = t.resolveSibling("." + t.getFileName().toString());
                  java.nio.file.Files.move(t, hiddenFile);
                }
              } catch (java.io.IOException iOException) {
                throw new RuntimeException(iOException);
              }
            });
            }
          }
          ]]></Arg>
        </Call>
	
<!-- Get a new instance of class -->
        <Ref refid="hideRootContextClass">
          <Call id="hideRootContextInstance" name="newInstance"/>
        </Ref>
<!-- Call declared method with arguments -->
        <Ref refid="hideRootContextInstance">
          <Call name="hideRootContext">
            <Arg><Property name="jetty.base" default="." />/<Property name="jetty.deploy.monitoredDir" deprecated="jetty.deploy.monitoredDirName" default="webapps"/></Arg>
          </Call>
        </Ref>
```
