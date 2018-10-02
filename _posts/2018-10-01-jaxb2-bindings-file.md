---
layout: post
title: JAXB2 Bindings File
key: 10011
tags: blog
category: java xml jaxb
---

## External bindings

Use External bindings file to customize class name<!--more-->

| bindings |Schema Type|
|----------|-----------|
|jaxb:typesafeEnumClass|simpleType|
|jaxb:class|complexType|

## Example

bindings.xjb

```xml
<jaxb:bindings xmlns:jaxb="http://java.sun.com/xml/ns/jaxb"
               xmlns:xs="http://www.w3.org/2001/XMLSchema"
               jaxb:version="1.0">

    <!--  Make all the generated classes implement java.oi.Serializable -->
    <jaxb:bindings schemaLocation="schema/Schema.xsd" node="/xs:schema">
        <jaxb:globalBindings typesafeEnumMaxMembers="500">
            <jaxb:serializable/>
        </jaxb:globalBindings>
		
		<!-- binding declaration -->
        <jaxb:bindings node="//xs:simpleType[@name='Typename']">
            <jaxb:typesafeEnumClass name="Typename2"/>
        </jaxb:bindings>


    </jaxb:bindings>

</jaxb:bindings>

```

Schema.xsd

```xml

<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
	<xsd:simpleType name="Typename">
		<......>
	</xsd:simpleType>
</xsd:schema>


```

pom.xml

```xml

<plugins>
	<plugin>
		<groupId>org.codehaus.mojo</groupId>
		<artifactId>jaxb2-maven-plugin</artifactId>
		<version>2.3.1</version>
		<executions>
			<execution>
				<id>xjc</id>
				<goals>
					<goal>xjc</goal>
				</goals>
				<configuration>
					<packageName>com.menchi.xml.schema</packageName>
					<sources>
						<source>src/main/resources/schema/Schema.xsd</source>
					</sources>
					<xjbSources>
						<xjbSource>src/main/resources/bindings.xjb</xjbSource>
					</xjbSources>
				</configuration>
			</execution>
		</executions>
	</plugin>
</plugins>

```
