You are a Java 21 + Spring Boot 3.2.x expert.

Create a complete, production-ready Spring Boot project from scratch called
"spring-boot-agents-demo" with the following exact structure and files.


══════════════════════════════════════════════════════════════════
SECTION 1 — PROJECT COORDINATES & STRUCTURE
══════════════════════════════════════════════════════════════════

GroupId    : javaone
ArtifactId : spring-boot-agents-demo
Version    : 1.0.0-SNAPSHOT
Java       : 21
Packaging  : jar

Package root: javaone.demo



══════════════════════════════════════════════════════════════════
SECTION 2 — pom.xml
══════════════════════════════════════════════════════════════════

Create pom.xml at the project root with:

Parent:
spring-boot-starter-parent 3.2.3

Properties:
java.version = 21
archunit.version = 1.3.0
checkstyle.version = 10.14.2
spotbugs.plugin.version = 4.8.3.1
maven.checkstyle.plugin.version = 3.3.1
jacoco.maven.plugin.version = 0.8.11

Dependencies:
- spring-boot-starter-web
- spring-boot-starter-data-jpa
- spring-boot-starter-validation
- com.h2database:h2 (runtime)
- org.projectlombok:lombok (optional)
- spring-boot-starter-test (test scope)
- com.tngtech.archunit:archunit-junit5:${archunit.version} (test scope)

Build plugins (all required, all configured):

PLUGIN 1 — spring-boot-maven-plugin
Exclude lombok from the final JAR.

PLUGIN 2 — maven-checkstyle-plugin ${maven.checkstyle.plugin.version}
Execution id   : checkstyle-validate
Phase          : validate
Goal           : check
configLocation : checkstyle.xml
consoleOutput  : true
failsOnError   : true
violationSeverity : warning
suppressionsLocation : checkstyle-suppressions.xml
Pin the checkstyle dependency to ${checkstyle.version}.

PLUGIN 3 — spotbugs-maven-plugin ${spotbugs.plugin.version}
Execution id   : spotbugs-verify
Phase          : verify
Goal           : check
effort         : Max
threshold      : Low
failOnError    : true
excludeFilterFile : spotbugs-exclude.xml
xmlOutput      : true
htmlOutput     : true

PLUGIN 4 — jacoco-maven-plugin ${jacoco.maven.plugin.version}
Exclusions:
javaone/demo/SpringBootAgentDemoApplication.class

Executions:
1. prepare-agent (default phase)
2. report         — phase: verify,  goal: report
3. check          — phase: verify,  goal: check
Rules:
BUNDLE / LINE   / COVEREDRATIO / minimum 0.80
BUNDLE / BRANCH / COVEREDRATIO / minimum 0.80

══════════════════════════════════════════════════════════════════
SECTION 3 — checkstyle.xml  (project root)
══════════════════════════════════════════════════════════════════

Create checkstyle.xml enforcing these rules exactly:

File-level checks (Checker module):
- FileTabCharacter      : eachLine=true  (no tabs)
- NewlineAtEndOfFile
- LineLength            : max=120, ignorePattern for package/import/URLs
- RegexpSingleline      : format=System\.(out|err)\.print  severity=error
- RegexpSingleline      : format=\s+$  (trailing whitespace)

TreeWalker checks:
Import rules:
- AvoidStarImport
- UnusedImports
- IllegalImport        : illegalPkgs=sun,com.sun

Naming:
- TypeName             : ^[A-Z][a-zA-Z0-9]*$
- MethodName           : ^[a-z][a-zA-Z0-9]*$
- LocalVariableName    : ^[a-z][a-zA-Z0-9]*$
- MemberName           : ^[a-z][a-zA-Z0-9]*$
- ConstantName         : ^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$
- PackageName          : ^[a-z]+(\.[a-z][a-z0-9]*)*$

JavaDoc:
- JavadocMethod        : scope=public, allowMissingParamTags=false,
allowMissingReturnTag=false,
tokens=METHOD_DEF,ANNOTATION_FIELD_DEF
- JavadocType          : scope=public
- JavadocVariable      : scope=public

Size / Complexity:
- MethodLength         : max=50, countEmpty=false
- CyclomaticComplexity : max=10
- ParameterNumber      : max=7

Coding style:
- NeedBraces           : IF,ELSE,FOR,WHILE,DO
- LeftCurly
- RightCurly
- MagicNumber          : ignoreNumbers=-1,0,1,2
ignoreAnnotation=true
ignoreHashCodeMethod=true
- EmptyBlock           : option=text, tokens=CATCH,FINALLY,IF,ELSE,TRY
- DefaultComesLast
- MissingSwitchDefault
- FallThrough
- EqualsHashCode
- HiddenField          : ignoreConstructorParameter=true, ignoreSetter=true
- StringLiteralEquality
- MultipleVariableDeclarations
- OneStatementPerLine
- ModifierOrder
- RedundantModifier

Whitespace:
- WhitespaceAround     : allowEmptyConstructors=true,
allowEmptyMethods=true, allowEmptyTypes=true
- NoWhitespaceBefore
- EmptyLineSeparator   : allowNoEmptyLineBetweenFields=true
tokens=IMPORT,STATIC_IMPORT,CLASS_DEF,
INTERFACE_DEF,ENUM_DEF,STATIC_INIT,
INSTANCE_INIT,METHOD_DEF,CTOR_DEF,
VARIABLE_DEF

══════════════════════════════════════════════════════════════════
SECTION 4 — checkstyle-suppressions.xml  (project root)
══════════════════════════════════════════════════════════════════

Suppress these rules:
- files=.*Test\.java$         checks=JavadocMethod
- files=.*Test\.java$         checks=JavadocType
- files=.*Test\.java$         checks=MagicNumber
- files=.*Test\.java$         checks=MethodLength
- files=SpringBootAgentDemoApplication\.java  checks=JavadocMethod

══════════════════════════════════════════════════════════════════
SECTION 5 — spotbugs-exclude.xml  (project root)
══════════════════════════════════════════════════════════════════

Exclude these known false positives with justification comments:
- EI_EXPOSE_REP2  for ~.*\.model\..*   (Lombok @Setter on JPA entities)
- EI_EXPOSE_REP   for ~.*\.model\..*   (Lombok @Getter on JPA collections)
- SE_BAD_FIELD    for ~.*Application   (Spring context refs not serializable)
- URF_UNREAD_FIELD for ~.*Configuration (Spring reflection-injected fields)
- RCN_REDUNDANT_NULLCHECK_OF_NONNULL_VALUE for ~.*Repository
- DM_DEFAULT_ENCODING for ~.*Test
- ST_WRITE_TO_STATIC_FROM_INSTANCE_METHOD for ~.*ArchitectureTest

══════════════════════════════════════════════════════════════════
SECTION 6 — src/main/resources/application.properties
══════════════════════════════════════════════════════════════════

spring.application.name=spring-boot-agents-demo

# H2 datasource
spring.datasource.url=
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# H2 console (dev only)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.open-in-view=false

# Jackson
spring.jackson.serialization.write-dates-as-timestamps=false
spring.jackson.default-property-inclusion=non_null

# Logging
logging.level.root=INFO
logging.level.javaone.demo=DEBUG
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

══════════════════════════════════════════════════════════════════
