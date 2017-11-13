# So you want to migrate a GWT 2 module? 

First, pick a module - we're [maintaining a list](https://docs.google.com/spreadsheets/d/1b1D9fEqRh5lZ8cqMJtYoc_25rfTRvsuJkTtS2vjgi3o/edit#gid=385798939)
that tries to explain how complex something is, how much work there is, what dependencies it has,
and who is interested in working on it. If the one you've picked has someone's name on it, contact
them to help, or you can always make your own version, for practice or just to have an alternate
approach.

The goal in porting these is to come up with a solution that allows existing GWT 2 projects to
take advantage of modern GWT features. This will let us continue to improve and update these 
modules without risking backward incompatibility, with the added bonus of enabling compatibility
with GWT 3, through the use of removing old JSNI and JavaScriptObjects in favor of JsInterop, and
rewriting Generators into Java's Annotation Processors.

Migration isn't easy for any project, so we want to keep this as simple as possible. To that end, 
as you make your changes to migrate the module, you'll want to release an initial version that is
as compatible with the old version as possible. We'll also want to be sure not to lose important
functionality, so it is important to also port the tests.

# Getting started

Suggested outline to get started:
* Create your new project, perhaps based on the example pom in this project (example gradle 
config would be helpful too - patches welcome!) Some highlights:
  * Uses the update gwt-maven-plugin, which appears to produce a library which can be compatible
  with J2CL.
  * Use the checkstyle plugin, with the checkstyle config here (copied from the GWT project for
  consistency).
  * Doesn't depend on gwt-user - if necessary, add dependencies on elemental or jsinterop-base,
  or other ported modules from the spreadsheet.
  * Update the groupId to your own domain if you intend to publish this. More on this below.
  * Set the module name in the gwt-maven-plugin.
* Copy the existing module into a new project, including its sources and tests. Build to ensure 
that everything is legal, and that the build correctly runs tests, produces Javadoc, etc. You'll
also need to update the `.gwt.xml` module file to behave with the maven plugin.
* Rename the package: usually replace `com.google.gwt` with `org.gwtproject` (and verify that tests 
pass)
* Build a sample project that use your newly renamed module, to ensure everything still works

Now we have enough that GWT2 projects should be able to update their imports, but still have
everything work. You might try this, to make sure that everything still behaves as expected.

# Update and improve
* Remove deprecated methods and classes - we’ve waited long enough, and we should start the new
version off with a clean slate.
* Check the module file, see if replace-with rules are needed, discuss ways to remove them. If you
need help with this, there are lots of good places to talk about it, including the [GWT 
Contributors mailing list](https://groups.google.com/forum/#!forum/google-web-toolkit-contributors),
and the [gwtproject/gwt Gitter channel](https://gitter.im/gwtproject/gwt).
* Change JavaScriptObject types to be native `@JsType`s. Constructors can now exist, but should still 
be called by their (now non-native) static factory methods, if any, to avoid breaking existing API
calls. At your discretion, you might consider deprecating those static factory methods.
* Start replacing simple JSNI methods either with plain Java, calling private native methods, or 
with just a native jsinterop method. Hopefully many of these cases will be obvious, and should
make the code more readable as you work. Again, don't be afraid to ask for help as you get started.
* Rewrite any required generators into annotation processors. This point seems short, but could span 
articles or a book - asking for help may not get you as much assistance here, but any module 
requiring generator work should be appropriately marked so you aren't surprised by this.

# API pitfalls
After all of this, you might find your APIs still depend on gwt-user for classes like
`JavaScriptObject` or `GWT`, preventing you from having GWT 3 compatible code. This is where things
get a little interesting: make sure there is a good alternative, and deprecate the old methods. Once
ready to release, publish this version of your library. Then, remove all of these deprecated methods,
and publish a newer version, without the dependency on gwt, or the old objects.

# Publish
Once you are in a stable state, let the community know so they can take a look! Of course, you can
just publish your sources somewhere like GitHub, but you can also push to a maven repository so
developers can download your work easily in their maven or gradle projects. The simple pom.xml
included in this project has the basics of how to push it to Sonatypes repository, but you will also
need to create a "groupId" for a domain name you control, or for your project within the `com.github.*`
namespace. See [Sonatype's guide](http://central.sonatype.org/pages/ossrh-guide.html) for more details.