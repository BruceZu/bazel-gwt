# GWT Rules

<div class="toc">
  <h2>Rules</h2>
  <ul>
    <li><a href="#gwt_application">gwt_application</a></li>
  </ul>
</div>

## Overview

These build rules are used for building [GWT](http://www.gwtproject.org/)
applications with Bazel. Applications are compiled as `.war` files containing
compiled JavaScript and other resources. GWT applications can also be run in
[Development Mode](http://www.gwtproject.org/doc/latest/DevGuideCompilingAndDebugging.html)
via `bazel run`.

<a name="setup"></a>
## Setup

To be able to use the GWT rules, you must provide bindings for the following
targets:

  * `//external:asm`
  * `//external:javax-validation`
  * `//external:java-validation-src`
  * `//external:gwt-dev`
  * `//external:gwt-user`

The easiest way to do so is to add the following to your `WORKSPACE` file:

```python
http_archive(
  name = "com_ekuefler_bazel_gwt",
  url = "https://github.com/ekuefler/bazel-gwt/archive/0.0.2.tar.gz",
  sha256 = "1e0abb048e2741f7bf57633d6562bb737911b0d831c907745313cc4b8e4a837e",
  strip_prefix = "bazel-gwt-0.0.2",
)
load("@com_ekuefler_bazel_gwt//gwt:gwt.bzl", "gwt_repositories")
gwt_repositories()
```

<a name="basic-example"></a>
## Basic Example

Suppose you have the following directory structure for a simple GWT application:

```
[workspace]/
    WORKSPACE
    src/main/java/
        app/
            BUILD
            MyApp.java
            MyApp.gwt.xml
        lib/
            BUILD
            MyLib.java
        public/
            index.html
```

Here, `MyApp.java` defines the entry point to a GWT application specified by
`MyApp.gwt.xml` which depends on another Java library `MyLib.java`. `index.html`
defines the HTML page that links in the GWT application. To build this app, your
`src/main/java/app/BUILD` can look like this:

```python
load("@com_ekuefler_bazel_gwt//gwt:gwt.bzl", "gwt_application")

gwt_application(
  name = "MyApp",
  srcs = glob(["*.java"]),
  resources = glob(["*.gwt.xml"]),
  modules = ["app.MyApp"],
  pubs = glob(["public/*"]),
  deps = [
    "//src/main/java/lib",
  ],
)
```

Now, you can build the GWT application by running
`bazel build src/main/java/app:MyApp`. This will run the GWT compiler and place
all of its output as well as `index.html` into
`bazel-bin/src/main/java/app/MyApp.war`. You can also run
`bazel run src/main/java/app:MyApp-dev` to run GWT development mode for the
application. Once development mode has started, you can see the app by opening
http://127.0.0.1:8888/index.html in a browser. Note that development mode assumes
that all of your `.java` files are located under `src/main/java/` - see details
on the `java_root` flag below if this is not the case.

For a complete example, see the
[`example/`](https://github.com/ekuefler/bazel-gwt/tree/master/example/src/main/java/com/ekuefler/sample)
directory in this repository.

<a name="gwt_application"></a>
## gwt_library

```python
gwt_application(name, srcs, resources, modules, pubs, deps, output_root, java_root, compiler_flags, compiler_jvm_flags, dev_flags, dev_jvm_flags):
```

### Implicit output targets

 * `<name>.war`: archive containing GWT compiler output and any files passed
   in via pubs.
 * `<name>-dev`: script that can be run via `bazel run` to launch the app in
   development mode.

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
      </td>
    </tr>
    <tr>
      <td><code>srcs</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          List of .java source files that will be compiled and passed on the
          classpath to the GWT compiler.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>resources</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          List of resource files that will be passed on the classpath to the GWT
          compiler, e.g. <code>.gwt.xml</code>, <code>.ui.xml</code>, and
          <code>.css</code> files.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>modules</code></td>
      <td>
        <code>List of strings, required</code>
        <p>
          List of fully-qualified names of modules that will be passed to the GWT
          compiler. Usually contains a single module name corresponding to the
          application's <code>.gwt.xml</code> file.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>pubs</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          Files that will be copied directly to the output war, such as static
          HTML or image resources. Not interpreted by the GWT compiler.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>deps</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          List of other java_libraries on which the application depends. Both the
          class jars and the source jars corresponding to each library as well as
          their transitive dependencies will be passed to the GWT compiler's
          classpath. These libraries may contain other <code>.gwt.xml</code>,
          <code>.ui.xml</code>, etc. files as <code>resources</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>output_root</code></td>
      <td>
        <code>String, optional</code>
        <p>
          Directory in the output war in which all outputs will be placed. By
          default outputs are placed at the root of the war file.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>java_root</code></td>
      <td>
        <code>String, optional</code>
        <p>
          Directory relative to the workspace root that forms the root of the
          Java package hierarchy (e.g. it contains a <code>com</code> directory).
          By default this is `src/main/java`. If your Java files aren't under
          <code>src/main/java</code>, you must set this property in order for
          development mode to work correctly. Otherwise GWT won't be able to see
          your source files, so you will not see any changes reflected when
          refreshing dev mode.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>compiler_flags</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          Additional flags that will be passed to the GWT compiler. See
          <a href='http://www.gwtproject.org/doc/latest/DevGuideCompilingAndDebugging.html#DevGuideCompilerOptions'>here</a>
          for a list of available flags.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>compiler_jvm_flags</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          Additional JVM flags that will be passed to the GWT compiler, such as
          <code>-Xmx4G</code> to increase the amount of available memory.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>dev_flags</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          Additional flags that will be passed to development mode. See
          <a href='http://www.gwtproject.org/doc/latest/DevGuideCompilingAndDebugging.html#What_options_can_be_passed_to_development_mode'>here</a>
          for a list of available flags.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>dev_jvm_flags</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          Additional JVM flags that will be passed to development mode, such as
          <code>-Xmx4G</code> to increase the amount of available memory.
        </p>
      </td>
    </tr>
  </tbody>
</table>
