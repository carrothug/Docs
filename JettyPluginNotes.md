#Note on (Jetty) Plugin for Pinpoint
---
#### Choose which version to support
Pinpoint uses Maven to build its projects, so we need to indicate which version of Jetty to use in the ```pom.xml``` file. Make sure that the Java version matches up. For example, Jetty's newest version 9.3 is for Java 8+, while Pinpoint still uses 6 or 7. The Jetty plugin uses version 9.2 of Jetty for this reason.

Problems with compiler/runtime version compatibility will look something like this:

	class file has wrong version 52.0, should be 50.0

#### Many different ways to intercept functions.
Take a look at [this sample](https://github.com/lioolli/pinpoint-plugin-sample) by lioolli. It has various examples of injectors along with comments on how they differ. Along with time stamps, we also keep track of relevant information from these RPCs for the user so that they have more knowledge at their disposal when analyzing problems.

My plugin keeps track of the remote IP address and http parameters in addition to the API, Client IP and Path as noted in by the red boxes. (Default loopback address in IPv6 is ```0:0:0:0:0:0:0:1```):![pinpoint screenshot](/Users/Jung/Desktop/Screen Shot 2015-07-17 at 3.56.22 PM.png) 

#### Finding an entry point
Most plugins don't need to differentiate between entry points and non-entry points. For Jetty (and Tomcat, or any other server/servlet plugin), however, finding entry points is crucial because Pinpoint needs to create a ```trace``` for each thread at an entry point. All other methods will be added to this trace to form a tree in the end.

Since the Jetty servlet is an implementation of the Java Servlet interface, I first considered intercepting that interface. This approach is not Jetty specific, therefore I don't have to worry about Jetty getting an upgrade and suddenly breaking my plugin. Also, this means that I might be able to use this plugin to cover all servlets that implement the Servlet interface. As one can see when inspecting the source code, what we do in the Tomcat plugin is not all that different from the Jetty plugin. Unfortunately, **Pinpoint can't profile interfaces.** The other option was to choose a method from the Jetty API. The next step was to analyze the stack traces of webapps that used Jetty to choose an entry point from Jetty methods.

Here we need to consider a couple more things:

- How familiar will Jetty users be with the method we choose. In other words, we want to pick a method that the user will be able to recognize as a Jetty entry point. If we do end up choosing a method that the user probably won't recognize, that's okay since Pinpoint shows which application and which library the method is from in the call stack.
- Each transaction needs to be profiled once. We don't want to miss any transactions, but we also don't want to profile the same one mutiple times, which means that we need to pick a set of methods that will cover all RPCs without overlapping. Since we need to differentiate between profiling synchronous requests and asychronous requests, I chose one entry point for each since the two methods are passed an argument that I can extract the HttpRequest from. The additional information we display for the user comes from the request. Another option would be to pick one method for all requests and differentiate between synchronous and asynchronous within the interceptor like the Tomcat plugin.

#### Other methods

Intercepting other methods are fairly straight forward from then on. There is no need to worry about overlap and coverage. We pick a method, figure out what kind of information you need in addition to transaction times and add them as shown in the samples. 

1

2

3

4

5

6

7

8

9










