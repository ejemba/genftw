genftw
======

JSR-269 framework for writing generators using templates and annotations. Generators For The Win.

this is a fork of http://code.google.com/p/genftw/ who is loooking inactive.


Generators For The Win
JSR-269 framework for writing generators using templates and annotations

GenFTW is a Java framework for addressing repeating patterns in software projects.

As each project grows and gains complexity over time, its source code or other files will usually reveal patterns that repeat throughout the project codebase. These can be technology or domain oriented, well-known or emerging, affecting code or other project artifacts. In most cases, maintaining such files by hand makes little sense.

With GenFTW, you can write generators that produce arbitrary files, such as additional source code or configuration. You can plug your generators into any Java tool that supports JSR-269 annotation processing API, such as the Java compiler itself. This means your handwritten source code can directly reference generated source elements. Your generators can use declarative element matching to produce their outputs based on for-all and for-each strategies. And you don't have to deal with JSR-269 API internals yourself, since GenFTW takes care of everything for you.

Let's take a look at a simple example. Suppose we want to generate JPA persistence.xml file that lists all managed persistence classes in our fictional project:

``` @javax.persistence.Entity public class Customer { // usual JPA entity stuff }

@javax.persistence.Entity public class Order { // usual JPA entity stuff } 
```

How much code does it take to write our custom generator?

Step one, generator descriptor (Java interface with some GenFTW annotations):

``` 
@Generator 
public interface JpaPersistenceXmlGenerator {

@Produces  ( output = "META-INF/persistence.xml", template = "com/myproject/templates/persistence.xml.ftl") 
@ForAllElements( 
   @Where(
      kind = ElementKind.CLASS, 
      annotations = "javax.persistence.Entity", 
      matchResultVariable = "entities"
   )
) 
void generatePersistenceXml();

} 
```

Step two, FreeMarker template (persistence.xml.ftl):
```
<persistence> 
  <persistence-unit name="OrderManagement"> 
    <!-- Managed persistence classes --> 
    <#list entities as e> 
      <class>${e.qualifiedName}</class> 
    </#list> 
</persistence-unit> </persistence>
```

And we're done! From now on, each time we build our project, the META-INF/persistence.xml file will be generated:

```
<persistence> 
  <persistence-unit name="OrderManagement"> 
    <!-- Managed persistence classes --> 
    <class>com.myproject.Order</class> 
    <class>com.myproject.Customer</class> 
  </persistence-unit> 
</persistence>
```

What happened behind the scenes when we built our project? 
* GenFTW annotation processor picked up our @Generator interface and analyzed its methods, looking for @Produces annotation 
* GenFTW element finder scanned all source elements of our project (Customer and Order classes), trying to satisfy @Where criteria that says:

    Give me all class elements that carry javax.persistence.Entity annotation, and provide them to the template as entities variable 
* GenFTW annotation processor loaded the associated FreeMarker template and processed it with regard to @Produces annotation values

Now this was quite a simple example to show you the basics. There's much more to GenFTW. Explore the wiki to learn what GenFTW can do, or get started right away.
