# Fragments - Best Practices and Useful Hints

A Fragment is a partial Accelerator and can do the same transformations that an Accelerator can do. However, it cannot run on its own, it’s always part of the calling (host) Accelerator.

Developing a Fragment is useful in the following situations:
- To update a version of an element of a technology stack in multiple locations. (e.g., the JDK version needs to be updated in the build tool configuration and the build pack configuration, and the deployment options)
- To add a consistent cross-cutting concern to a set of Accelerators, such as logging, monitoring, or support for a certain type of deployment or support for a certain framework.
- To add integration with some technology to a generated application skeleton. For example, certain database support or support for a certain messaging middleware or an integration with an email provider.

## <a id="design-considerations"></a> Design considerations

Developing and maintaining a Fragment is complex:

- As a Fragment author you don’t know what kind of formatting rules or syntax variation is used in a host Accelerator. A Fragment you develop must be ready to work with all possible syntax and format variations. For example, dependency in a `Gradle` build.gradle.kts can have the following forms:

    - `implementation(‘org.springframework.boot:spring-boot-starter’)`
    - `implementation("org.springframework.boot:spring-boot-starter")`
    - `implementation(group = "org.springframework.boot”, name= “spring-boot-starter")`
    - `implementation(group = ‘org.springframework.boot’, name= ‘spring-boot-starter’)`
    - `implementation(name= “spring-boot-starter", group = "org.springframework.boot”)`

- The Fragment is used in multiple Accelerator contexts and behavior still results in a compiled and deployable Application Skeleton
- As a testing, a Fragment in isolation is more difficult than testing an Accelerator, testing takes more time as all the combinations need to be tested from an Accelerator perspective.

Some considerations:

- To be reused flexibly and in different combinations, each fragment must cover a small, cohesive piece of functionality. In other words, fragments must follow these two UNIX principles:

    - Small is Beautiful
    - Each Program (Fragment) Does One Thing Well

- Keep the files it changes to a minimum: only change the files that are related to the same technology stack for the same purpose.
- Be mindful that the design of both the Accelerator and the Fragment is naturally limited by the technology stack and target deployment technology that is being chosen for the Accelerator. For example, if you need to create a Fragment for standardizing logging, create one per base technology stack.

## <a id="housekeeping"></a> Housekeeping rules

Fragments are used by Accelerator Authors. VMware has found that the following guidelines keep our Fragments understandable and able to be reused.

- Fragments must have an intuitive name and short description that reflects their purpose. The name must not include the word ‘fragment’.
- Fragments must expose options to allow configuring the output of execution.
- As the main and the only users of fragments are accelerator authors, each fragment must contain a README explaining what additional functions this fragment adds to a generated application skeleton and what options are expected by this fragment. It must contain a description of how this fragment may be included in a host accelerator. If there are any known limitations or not covered use cases, they must be clearly stated in the README (e.g., fragment supports Maven and Gradle as build tools but only Groovy DSL of Gradle is supported).
- If a fragment must provide any additional documentation to end users it can either add a README-X file to the generated application skeleton or append a section to the host’s README.
