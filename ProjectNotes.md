# Hands-on steps
After cloning/forking the project: 

./pull-dependencies **core corenlp freebase tables**

retrieves the jar libraries required to work with the source code: 

- **core** retrieves the libraries required by the core functionality of sempre
- **corenlp** retrieves the coreNLP libraries 
- **freebase** retrieves the freebase module libraries (lucene 4.4 among others)
- **table** retrieves opencsv3.0.jar 

All of them are required to solve the dependency problems. Apart from that, fig.jar by itself does not include any compiled class, but just the source code. There is a *fig* subfolder (it corresponds to another github project), where we can find all the code required. In eclipse, the cleanest way to make it work is to include fig/src/main/java as src folder. The libraries required to compile it are in *fig/external*. In this case, I've copied them. 

To make it run from commandline, it has to be built with the ant script it contains. We have to modify it a little to include the src directory of fig in the compilation process: 

```xml 
<property name="figSRC" location="${root}/fig/src/main/java/"/>
...
<javac srcdir="${src}:${figSRC}"
```

In runtime it still requires the .jar file, so we can build directly the subproject fig using its ant built.xml file and copy it to lib. To do so, we modify its ant file: 

```
<project default="compile">
  <target name="compile" >
    <path id="lib.path">
    	<fileset dir="external"/>
	</path>
    <javac srcdir="src/main/java" destdir="classes" classpathref="lib.path" debug="true"/>
  	<jar destfile="ourFig.jar" basedir="classes" includes="fig/**"/>
  </target>
</project>
```

Voilà!!

# Isolating steps

According to Panupong, the quickest way would be to change CanoninalNames. But it seems that it is not the case due to the type inferring mechanisms. 

We start then using just the simple-sparql mode, which uses the lambda-DCS to SPARQL translator, to build a final SPARQL query. 

* **SPARQLExecutor**

	* Getting rid of all the references to freebase and adding the prefixes **(hardcoded so far)**, it seems to work. 
	* Also required to modifiy *SparqlExpr* to get rid of all the well-formed checkings. I-ve just let everything pass. **Far better way**: check whether it is a well-formed IRI, and re-enable the rest of checkings. 
	* *NodeToValue* has to be modified to provide better typing feedback ... currently all this information is lost, and everything comes again without any information. 
	* *FreebaseInfo* should be modified to have information about the ontology in a more generic way. **Conversion to SchemaInfo?** Using an OWL ontology to get all such information using the OWLAPI is not so difficult and would make a difference. Different possible sources: hardcoded freebase schema source, ontologies, SPARQL endpoint self-check (SPARKLIS-like approach). 

