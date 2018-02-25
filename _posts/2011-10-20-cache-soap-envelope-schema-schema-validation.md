---
layout: post
title: Cache Soap envelope schema for schema validation
categories:
- Linux
tags:
- soap
- xml
- bash
---
I recently had to enable schema validation on incoming requests on a Soap API running on Zend Soap. We were having noticeably worse performance with schema validation than without - and found out that it was because of the import tag pulling in the Soap envelope schema for every request:

{% highlight xml %}
<xsd:import namespace="http://schemas.xmlsoap.org/soap/envelope/" schemaLocation="http://schemas.xmlsoap.org/soap/envelope/"/>
{% endhighlight %}

My first thought was to match all the schemaLocations in the schemas and cache them manually, but it seemed to me there must be a better way..
There is and it's called a catalog.

"What's a catalog" I hear you say:

> Basically it's a lookup mechanism used when an entity (a file or a remote resource) references another entity. The catalog lookup is inserted between the moment the reference is recognized by the software (XML parser, stylesheet processing, or even images referenced for inclusion in a rendering) and the time where loading that resource is actually started.
[xmlsoft.org](http://xmlsoft.org/catalog.html)

So, it turns out libxml has catalog support. A catalog is basically an XML file that libxml will parse and use to map references. I was doing the schema validation with
[*DomDocument::schemaValidate*](http://php.net/manual/en/domdocument.schemavalidate.php) and since DomDocument uses libxml behind the scenes, this works for PHP as well. libxml2 per default looks for an xml catalog in */etc/xml/catalog* and DomDocument is hardwired to that location as well.

You can add XML catalogs with a little tool called *xmlcatalog* that comes with libxml (I think). The usage is pretty straightforward, call up the man page to get an overview or [read it online](http://xmlsoft.org/xmlcatalog_man.html). Here’s how I added a catalog to map the Soap envelope schema location to a local path:

1 - Create */etc/xml* if it doesn't yet exist.
2 - Copy the Soap envelope schema from [http://schemas.xmlsoap.org/soap/envelope/](http://schemas.xmlsoap.org/soap/envelope/) to a local path, let’s say */etc/xml/soap-envelope-1.1.xsd*
3 - Create the catalog file (if it doesn’t exist yet) and add our new rule:

{% highlight bash %}
xmlcatalog --create --noout --add "rewriteURI" "http://schemas.xmlsoap.org/soap/envelope/" \
"file:///etc/xml/soap-envelope-1.1.xsd" \
/etc/xml/catalog
{% endhighlight %}

The file created at */etc/xml/catalog* should then look something like:

{% highlight xml %}
<?xml version="1.0"?>
<!DOCTYPE catalog PUBLIC "-//OASIS//DTD Entity Resolution XML Catalog V1.0//EN" "http://www.oasis-open.org/committees/entity/release/1.0/catalog.dtd">
<catalog xmlns="urn:oasis:names:tc:entity:xmlns:xml:catalog">
 <rewriteURI uriStartString="http://schemas.xmlsoap.org/soap/envelope/" rewritePrefix="file:///etc/xml/soap_1.1_envelope.xsd"/>
</catalog>
{% endhighlight %}

4 - Restart apache

Your schema validation should be quick as a flash after that.

\* You can change this default location by setting the *XML_CATALOG_FILES* environment variable.
