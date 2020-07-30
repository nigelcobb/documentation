# CDM Distribution - How to

This guide is a collection of short how-to code guides for common CDM Distribution questions.

* How to open import maven dependencies
* How to generate Global Keys & Qualify instance
* How to validate a CMD Java Instance


## JAVA

### Pre reqs

* Java SDK 11

### Introduction

* The CDM distribution is build using [maven](https://maven.apache.org) and is published using the REGnosys's artifactoy. This can be downladed from the [CMD Portal](https://isda:isda@regnosys.jfrog.io/regnosys/libs-snapshot/com/isda/cdm-distribution/2.64.0/cdm-distribution-2.64.0.zip).

* There is also an example project (CDM Examples), which shows how the examples can be run. See video tutorial [here](https://vimeo.com/359012532) and to download click [here](https://isda:isda@regnosys.jfrog.io/regnosys/libs-snapshot/com/regnosys/isda-cdm-examples/2.64.0/isda-cdm-examples-2.64.0.zip)

*  The model objects are classified into namespaces (cdm.base, cdm.base.staticdata etc). This translates into same Java packages, with each package containing package-info file.

* The CDM uses [builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) for each of the pojo. The distribution ships with the json to java object serialisers

### How To: Setup Google's Guice Injector

CDM uses [Google's Guice](https://github.com/google/guice), as a dependency manager. Injector is the core of Guice that contains the whole object
graph (context).

So the first step is to initialise this injector. There are 2 options:

#### Option 1: (Using provided CdmRuntimeModule)

The CDM distribution comes with a pre-build CDM module that you can use to create an injector.

``` Java
    Injector injector = Guice.createInjector(new CdmRuntimeModule()));
```

#### Option 2: (Build your own Module)

In case you want to build the injector not based on the CDM's runtime module, then you can create your own. First create a Guice module with at least the following 2 bindings, as follows:

``` Java
  public class GenericModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(ModelObjectValidator.class).to(RosettaTypeValidator.class);
        bind(QualifyFunctionFactory.class).to(QualifyFunctionFactory.Default.class);
    }
  }
```

Once you have your own module, you can use it to create the injector.

``` Java
    Injector injector = Guice.createInjector(new GenericModule()));
```
### How To: Generate Global Keys and Qualifications

With in the model anything marked with metadata key will have a global key. These global keys are automicatically generated using the hash algorithms using the below code.

In order to post process the model objects with Global Keys and Qualify then you can follow the steps below:

Using the injector created in the previous step, it's easy to generate Global Keys and Run Qualifications.

``` Java
    Contract cdmInstance = buildCdmInstance();// The object that you build
    Contract.ContractBuilder builder = cdmInstance.toBuilder();
    keyProcessor.runProcessStep(Contract.class, builder);
    Contract updatedCdmInstance = builder.build();
```

### How To: Validate the CDM instance

In order to validate the CDM instance, you have to create a RosettaTypeValidator and post process the instance as follows:

``` Java
    RosettaTypeValidator validator = injector.getInstance(RosettaTypeValidator.class);
    ValidationReport validationReport = validator.runProcessStep(cdmInstance.getClass(), cdmInstance.toBuilder());
    if (validationReport.success()) {
     // handle failures
         List<ValidationResult<?>> validationResults = validationReport.validationFailures();
    }
```
If the validation is successful then the validation results object will contain the list of all the validation failures.
