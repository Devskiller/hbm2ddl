[![Build Status](https://travis-ci.org/Devskiller/hbm2ddl-maven-plugin.svg?branch=master)](https://travis-ci.org/Devskiller/hbm2ddl-maven-plugin)   [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.devskiller.hbm2ddl-maven-plugin/hbm2ddl-maven-plugin/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.devskiller.hbm2ddl-maven-plugin/hbm2ddl-maven-plugin)

# Hibernate Schema Generator Plugin

## Motivation

Why another tool to dump JPA schema? All tools that we've found were related to legacy versions of Hibernate or were covering just a simple cases, without option to configure dialect or naming strategy. Also all tools we found are based on the `SchemaExport` class, which does not always correlate with the runtime schema - for example due to the lack of support for the `Integrator` services, used to register `UserType` classes like JodaTime or similar. We were also looking for a tool that will be able to handle further schema migrations, but just dump the current version. 

## Usage

You can run this plugin directly or integrate it into the default build lifecycle.

### Simple configuration example

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.devskiller.hbm2ddl-maven-plugin</groupId>
            <artifactId>hbm2ddl-maven-plugin</artifactId>
            <version>0.9.2</version>
            <extensions>true</extensions> <!-- required to run automatically -->
            <configuration>
                <packages>
                    <package>com.test.model</package>
                </packages>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Configuration parameters

- `packages` (required): list of packages containing JPA entities
-  `outputPath`: output file for the generated schema. By default:
  - for `UPDATE` action: `${project.build.directory}/generated-resources/scripts/`
  - for other actions: `${project.build.directory}/generated-resources/scripts/database.sql`
- `jpaProperties`: additional properties like dialect or naming strategies which should be used in generation task. By default `empty`
- `formatOutput`: should the output be formatted. By default `true`
- `delimiter`: delimiter used to separate statements. By default `;` 
- `action`: which statements should be generated. By default: `CREATE`. Possible values:
  - `DROP`
  - `CREATE`
  - `DROP_AND_CREATE`
  - `UPDATE`
- `generationMode`: schema generation mode. By default `DATABASE`. Possible values:
  - `DATABASE`: generation based on setting up embedded database and dumping the schema
  - `METADATA`: generation based on static metadata

### Generate schema

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.devskiller.hbm2ddl-maven-plugin</groupId>
            <artifactId>hbm2ddl-maven-plugin</artifactId>
            <version>0.9.2</version>
            <configuration>
                <outputPath>${basedir}/src/main/resources/database.sql</outputPath>
                <packages>
                    <package>com.test.model</package>
                    <package>com.test.entities</package>
                </packages>
                <jpaProperties>
                    <property>
                        <name>hibernate.dialect</name>
                        <value>org.hibernate.dialect.MySQL57Dialect</value>
                    </property>
                    <property>
                        <name>hibernate.default_schema</name>
                        <value>prod</value>
                    </property>
                </jpaProperties>
                <formatOutput>true</formatOutput>
                <delimiter>;</delimiter>
                <action>DROP_AND_CREATE</action>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Generate migrations

It's also possible to generate automated migrations scripts. JPA2DDL supports [Flyway naming patterns](https://flywaydb.org/documentation/migration/sql) for versioned migrations.

All subsequent migration scripts are saved in the `outputPath`, in the following layout:
```sh
src/main/resources/migrations/
 v1__jpa2ddl.sql
 v2__jpa2ddl.sql
 ... next
```

Sample configuration:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.devskiller.hbm2ddl-maven-plugin</groupId>
            <artifactId>hbm2ddl-maven-plugin</artifactId>
            <version>0.9.2</version>
            <configuration>
                <outputPath>${basedir}/src/main/resources/migrations/</outputPath>
                <packages>
                    <package>com.test.model</package>
                    <package>com.test.entities</package>
                </packages>
                <jpaProperties>
                    <property>
                        <name>hibernate.dialect</name>
                        <value>org.hibernate.dialect.MySQL57Dialect</value>
                    </property>
                </jpaProperties>
                <formatOutput>true</formatOutput>
                <delimiter>;</delimiter>
                <action>UPDATE</action>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Direct invocation

```
./mvnw com.devskiller.hbm2ddl-maven-plugin:hbm2ddl-maven-plugin:0.9.2:generate
```