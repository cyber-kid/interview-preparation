# Class Loaders
Types of Built-in Class Loaders: 
* application;
* extension;
* bootstrap;
 
The **application class loader** loads the class where the example method is contained. An **application or system class loader** loads our own files in the classpath. Next, the **extension** one loads the Logging class. **Extension class loaders** load classes that are an extension of the standard core Java classes. Finally, the **bootstrap** one loads the ArrayList class. A **bootstrap or primordial class loader** is the parent of all the others.

However, we can see that the last out, for the ArrayList it displays null in the output. This is because the bootstrap class loader is written in native code, not Java – so it doesn’t show up as a Java class. Due to this reason, the behavior of the bootstrap class loader will differ across JVMs.

When the JVM requests a class, the class loader tries to locate the class and load the class definition into the runtime using the fully qualified class name. The **java.lang.ClassLoader.loadClass()** method is responsible for loading the class definition into runtime. It tries to load the class based on a fully qualified name. If the class isn’t already loaded, it delegates the request to the parent class loader. This process happens recursively. Eventually, if the parent class loader doesn’t find the class, then the child class will call **java.net.URLClassLoader.findClass()** method to look for classes in the file system itself. If the last child class loader isn’t able to load the class either, it throws **java.lang.NoClassDefFoundError** or **java.lang.ClassNotFoundException**.
```java
protected Class<?> loadClass(String name, boolean resolve)
  throws ClassNotFoundException {
     
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
 
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
 	
JIT compiler works on JVM to transform bytecode to native code for performance optimization.