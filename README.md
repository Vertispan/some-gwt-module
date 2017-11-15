# So you want to migrate a GWT 2 module? 

First, pick a module - we're [maintaining a list](https://docs.google.com/spreadsheets/d/1b1D9fEqRh5lZ8cqMJtYoc_25rfTRvsuJkTtS2vjgi3o/edit#gid=385798939)
that tries to explain how complex something is, how much work there is, what dependencies it has,
and who is interested in working on it. If the one you've picked has someone's name on it, contact
them to help, or you can always make your own version, for practice or just to have an alternate
approach.

The goal in porting these is to come up with a solution that allows existing GWT 2 projects to
take advantage of modern GWT features, without relying on GWT 3 incompatible features.
Specifically, JSNI, Generators and JavaScriptObject will not work in GWT 3; we must instead use
JsInterop and Java Annotation Processors (or other code generator / build tools).
By doing this porting now, we can improve and update these modules without risking backward incompatibility,
plus we can get the features most existing applications use ready for GWT 3, so we can release GWT 3 sooner,
with more confidence that most of the community can upgrade smoothly!

Migration isn't easy for any project, so we want to keep this as simple as possible. To that end, 
as you make your changes to migrate the module, you'll want to release an initial version that is
as compatible with the old version as possible. We'll also want to be sure not to lose important
functionality, so it is important to also port the tests.

# Getting started

Suggested outline to get started:
* Create your new project, perhaps based on the example pom in this project (example gradle 
config would be helpful too - patches welcome!) Some highlights:
  * Uses the updated gwt-maven-plugin, which can produce libraries that are compatible with J2CL (if you don't use JSNI / JavaScriptObject / Generators, of course).
  * Use the checkstyle plugin, with the checkstyle config [here](https://github.com/Vertispan/some-gwt-module/blob/master/gwt-checkstyle.xml) (copied from the GWT project for
  consistency).
  * Does _not_ depend on gwt-user - if necessary, add dependencies on elemental 2 or jsinterop-base,
  or other ported modules from the spreadsheet.
  * Update the groupId to your own domain if you intend to publish this. More on this below.
  * Set the module name in the gwt-maven-plugin (or manually maintain your own gwt.xml if using other build tools).
* Copy the existing module into a new project, including its sources and tests. Build to ensure 
that everything is legal, and that the build correctly runs tests, produces Javadoc, etc. You'll
also need to update the `.gwt.xml` module file to behave with the maven plugin. **(TODO: examples?)**.
* Rename the package: usually replace `com.google.gwt` with `org.gwtproject` (and verify that tests 
pass).
* Build a sample project that use your newly renamed module, to ensure everything still works.

Now we have enough that GWT2 projects should be able to update their imports, but still have
everything work. You should try this, to make sure that everything still behaves as expected.

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
* **(James' rewrite of above paragraph:)** Replace JSNI methods.  Where possible, use plain java by creating JsInterop types and methods
to be able to call into javascript.  While it is possible to add native javascript in J2CL, the new
`native.js` strategy cannot call back into java, and should be considered a last resort. Hopefully most of
the removal of JSNI should be straighforward, obvious, and make the code more readable as you work.
Again, don't be afraid to ask for help or code review as you get started.
* Rewrite any required generators into annotation processors, or other reusable generator tools.
This point seems short, but could span articles or a book - asking for help may not get you as
much assistance here, but any module requiring generator work should be appropriately marked
so you aren't surprised by this.

# API pitfalls
After all of this, you might find your APIs still depend on gwt-user for classes like
`JavaScriptObject` or `GWT`, preventing you from having GWT 3 compatible code. This is where things
get a little interesting: First, come join us on the gwt contributor mailing list or gitter to
discuss your problem.  If there is a good solution you can use now, the other contributors can
help you find it.  If the only alternative is to make a breaking change with GWT 2, then deprecate
anything using the old, GWT-2-only methods, and once ready to release, publish this version of your
library (with a version < 1.0). Then, remove all of these deprecated methods,
and publish a newer version, without the dependency on gwt, or the old objects.

# Publish
Once you are in a stable state, let the community know so they can take a look! Of course, you can
just publish your sources somewhere like GitHub, but you can also push to a maven repository so
developers can download your work easily in their maven or gradle projects. The simple pom.xml
included in this project has the basics of how to push it to Sonatype's repository, but you will also
need to create a "groupId" for a domain name you control, or for your project within the `com.github.*`
namespace. See [Sonatype's guide](http://central.sonatype.org/pages/ossrh-guide.html) for more details.
