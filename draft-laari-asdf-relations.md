---
title: "Extended relation information for Semantic Definition Format (SDF)"
abbrev: "SDF Relations"
docname: draft-laari-asdf-relations-latest
category: std

ipr: trust200902
area: ART
workgroup: ASDF Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: Petri Laari
    organization: Ericsson
    email: petri.laari@ericsson.com

normative:
  SDF: I-D.ietf-asdf-sdf
informative:
  DTDL:
    target: https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md
    title: Digital Twins Definition Language (DTDL) v2
    date: 2022-02-10
  saref4bldg:
    target: https://saref.etsi.org/saref4bldg
    title: SAREF extension for building
    date: 2020-06-05
    author:
      - name: María Poveda-Villalón
      - name: Raúl Garcia-Castro



--- abstract

The Semantic Definition Format (SDF) base specification defines set of basic information elements that can be used for describing a large share of the existing data models from different ecosystems. While these data models are typically very simple, such as basic sensors definitions, more complex models, and in particular bigger systems, benefit from ability to describe additional information on how different definitions relate to each other. This document specifies an extension to SDF for describing complex relationships in class level descriptions. This specification does not consider instance-specific information.

--- middle

# Introduction

The Semantic Definition Format (SDF) {{SDF}} is a format for domain experts to use in the creation and maintenance of data and interaction models in the Internet of Things. The SDF specification defines a generic data model that can be used as a meta model when converting between other data models, such as IPSO Smart Objects or Digital Twins Definition Language (DTDL) {{DTDL}}. SDF model defines a set of affordances, describing the interfaces for the Object. These can be mapped to corresponding affordances in other data models.

The base specification defines ways to represent parent-child relations between two definitions. However, sometimes there is a need to describe also more complex relations to support arbitrary connections between definitions and also referring to definitions outside of the SDF models. These could be, for example, defining possible location of a device inside a room, how a device is controlled by another device, or physical topology between devices. This enables defining more complex systems using SDF models.

The basic parent-child relations between SDF Objects and Things can be defined by including a definition of a child in the definition of the parent. This covers a large share of simple data models defining, e.g., simple sensors, or more complex devices containing a set of sensors. On the other hand, SDF can be used also to describe even more complex entities, such as buildings with rooms and other related objects inside a building. When we extend the SDF usage, the simple parent-child relation is often not enough, but more complex relations may be needed to describe the connections between the definitions. These relations can be for example physical (e.g., an object is inside another object), functional (e.g., an object can control another object), or semantic (e.g., an object is similar to a term defined in another ontology).

This document extends the base SDF specification by adding a new keyword to describe also other relations between physical or logical objects than plain parent-child relations. This new keyword is needed to describe, without loss of information, models from ecosystems that are using complex relation information in their definitions.

This extension enables describing relations from SDF models to various (SDF or other) definitions. For a link data type for affordances, e.g., for a link property that can be accesses and modified during runtime, the "sdfType for links" extension {{?I-D.bormann-asdf-sdftype-link}} can be used instead.

NOTE: This extension is now defined based on the Relationships feature in the DTDL specification. There may be other kind of definitions for relationships in other data models that must be taken into account and this specification may need to be extended to cover also those requirements.

# Terminology

This specification uses the terminology specified in {{SDF}}, in particular "Class Name Keyword", "Object", and "Affordance".

{::boilerplate bcp14-tagged}

# SDF Relation Extension

This section defines a new SDF Class Name Keyword, sdfRelation, that can be used to describe complex relations. The relationship definitions are on class-level, i.e., the sdfRelation keyword does not describe instance specific information about the relation, but describes how different models and definitions relate to each other.

## Namespaces

The SDF namespace block can be used to provide CURIE prefixes for external ontologies for use with sdfRelation extension. For example, in case of SAREF (Smart Applications REFerence ontology) ontology extension for buildings {{saref4bldg}}, we can use the following namespace definition:

<sourcecode>
{
  "namespace": {
    "saref": "https://saref.etsi.org/saref4bldg/v1.1.2/"
  }
}
</sourcecode>



## Qualities of sdfRelation

In this section, the qualities of the sdfRelation are defined. These qualities are used to define the potential type of the connection between the definitions and to which definition the connection can be made.

| Quality     | Type        | Required | Description                                      |
| ----------- | ----------- | -------- | ------------------------------------------------ |
| relType     | string/IRI? | no       | What kind of relationship these definitions have |
| target      | string      | no       | Target model for the relation                    |
| description | string      | no       | Description of the relationship                  |
| label       | string      | no       | Short text describing the relationship           |
| property    | object      | no       | Additional properties for this relation / this is inherited from DTDL, not valid for SDF    |
| $comment | string | no | Additional comments for implementors |

### relType

The relType quality describes what kind of relationship this definition has to the target definition. This can use different ontologies, such as SAREF from ETSI. The used ontology MUST be defined in the namespace block to give a short name for the ontology IRI.

For example the "relType" field could define the relationship to be `saref:isControlledByDevice`, when the SAREF ontology is used with CURIE prefix "saref" defined in the namespace block for the full IRI `https://saref.etsi.org/saref4bldg/v1.1.2/`. The defined purpose for the relation is a functional relationship between the two definitions.

### target

The "target" field defines to which definition or ontology term this definition with sdfRelation has a relation to. For example, this can be `#/sdfObject/room`, when the target object `room` is defined in the same SDF model. This may also be left undefined, and in that case the relation may be any other object (Note: This is from DTDL (check), does it make sense in SDF context?)

In addition to SDF definitions, the target can be also a reference to another ontology. For example, a temperature sensor SDF definition can be augmented with information that a SAREF definition of a TemperatureSensor has similar semantics as this SDF definition.

<sourcecode>
{
  "namespace": {
    "exont": "https://example.com/relationOntology",
    "saref": "https://saref.etsi.org/core/v3.1.1/"
  },
  "sdfObject": {
    "temperature": {
      "description": "Example temperature object",
      "sdfProperty": {
        ...
      },
      "sdfRelation": {
        "sameAs": {
          "relType": "exont:same-as",
          "target": "saref:TemperatureSensor"
        }
      }
    }
    ...
</sourcecode>


### description

The description of the relationship. For SDF version 1.1, the description is a string. (For future SDF versions this description can be localizable, allowing different languages in the description.)

### label

Short text describing the relationship, similar to label quality in other SDF definitions.

## Example relation description with sdfType links

In the following example, we have a definition for `first-object` which located next to `second-object`:

<sourcecode>
{
  "namespace": {
    "exont": "https://example.com/relationOntology"
  },
  "sdfObject": {
    "first-object": {
      "description": "Example object",
      "sdfProperty": {
        "adjacent-node": {
          "type": "object",
          "sdfType": "link"
        }
      },
      "sdfRelation": {
        "next-to": {
          "description": "This object is adjacent to the second
                          object",
          "relType": "exont:next-to",
          "target": "#/sdfObject/second-object"
        }
      }
    },
    "second-object": {
      "description": "Example object, next to the first object",
      "sdfProperty": {
        "adjacent-node": {
          "type": "object",
          "sdfType": "link"
        }
        ...
      },
      "sdfRelation": {
        "next-to": {
          "relType": "exont:next-to",
          "target": "#/sdfObject/first-object"
        }
      }
    }
  }
</sourcecode>

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.

# Appendix A. Formal Syntax of sdf-relation

<sourcecode>
sdfRelation = ( sdfRelation: {
  ? relType: global
  ? target: global
  ? description: text
  ? label: text
  ? property: { * text => text }
  ? $comment: text
})

; from SDF CDDL
global = text .regexp ".*[:#].*" ; rough CURIE or JSON Pointer syntax
</sourcecode>

# Appendix B. Ecosystem Mappings

This appendix provides an example mapping for SDF relations to related
approaches from selected IoT ecosystems.

TBD: This appendix will probably be removed eventually.

## DTDL - SDF conversion

This section (to be removed) discusses the mapping between SDF and DTDL qualities.

| Quality (DTDL) | Quality (SDF) | Description | Required in DTDL |
| @type | sdfRelation | DTDL Interface (Relationship), maps to sdfRelation in SDF | yes |
| @id | - | DTDL: The ID of the relationship description | no |
| comment | $comment | 	A comment for model authors | no |
| description | description | DTDL: localizable description for display | no |
| displayName | label | DTDL: localizable name for display | no |
| maxMultiplicity | - | max multiplicity for the target, maps to maxItems in SDF instance | no |
| minMultiplicity | - | min multiplicity for the target (must be zero), maps to minItems in SDF instance | no |
| name | "name of relation" | The programming name of the element | yes |
| properties | to sdfProperty | A set of Properties that define Relationship-specific state | no |
| target | target | An interface identifier of the target (or "any" if not specified) | no |
| writable | - | 	A boolean value that indicates whether the Relationship is writable or not, maps to SDF instance "writable" | no |


### DTDL specific conversion

#### DTDL @type and DTDL name

This defines the sdfRelation itself and the name is the name of the sdfRelation entry, i.e. @type Relationship and name converts to:

<sourcecode>
...
"sdfRelation": {
  "name-from-DTDL": {
    ...
  }
}
</sourcecode>


#### DTDL @id

In the example DTDL files, this is never present. This is the identifier for the relationship, no further definition in the specification. In DTDL this value is given automatically if it does not exist in the DTDL model file.

#### DTDL comment

This can be converted to $comment and it is a comment for the implementors.

#### DTDL description

This maps directly to the SDF "description".

#### DTDL displayName

This converts to the "label" field in SDF.

#### max and minMultiplicity

These define how many instances of the relationship can exist of the target type. The sdfRelation is purely a class-level definition, but sdfType "link" defines the actual instance specific information. Thus, these fields map to maxItems and minItems in the corresponding sdfType "link" definition.

#### DTDL properties

Relationship definition in DTDL may contain additional properties (key-value pairs) that describe additional properties for this relationship. This can be converted into sdfProperty in the same object as where the sdfRelation definion is.

#### DTDL target

In DTDL this is the Interface of the target, in SDF this maps to the target object of this relation.

#### DTDL writable

The relationship itself is not defined to be writable, but this field maps to the SDF instance and to the  corresponding sdfType "link" definition.

#### SDF Relation type

In SDF, the relType is giving the type of the relationship, e.g. isControlledBy. However, in DTDL, this is not directly described in the DTDL file.

--- back

# Acknowledgments
{:numbered="false"}

The author wants to thank Ari Keranen, Mikko Saarisalo, and Christer Holmberg for their feedback and comments.
