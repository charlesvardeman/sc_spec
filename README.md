# Smart Containers

[![Join the chat at https://gitter.im/charlesvardeman/sc_spec](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/charlesvardeman/sc_spec)

## Table of contents

 1. [Quick intro](#quick-intro)
 2. [Brief example of Smart Containers in action](#brief-example-of-solid-in-action)
 3. [RDF](#rdf)
 4. [Ontologies](#Ontologies)
 4. [Reading and writing data using LDP](#reading-and-writing-data-using-ldp)
 5. [Reading and writing data using SPARQL](#reading-and-writing-data-using-sparql)
 8. [Identity management](#identity-management-based-on-webid)
 9. [Personal data workspaces](#personal-data-workspaces)
 10. [Authentication](#authentication)
 11. [Access control](#access-control)
 12 [Software implementing SmartContainers](#software-implementing-solid)

Specification for Smart Containers

## Quick Intro
Scientists need an efficient approach to preserve and share the results of computational work and in fact, is required by reproducibility requirements of the scientific process. Although a variety of technologies have been created to capture computational experiments, it is still a struggle for scientists to understand the preserved experiment without a detailed documentation and context of the results being shared. Smart Container (SC) is a modular specification and implementation that leverages linked data principles to enable the preservation, sharing and reuse of both software and data artifacts.

Smart Containers begin with conceptualizing computational experiments from the perspective of computational environments and activities within those environments as provided by the Docker Linux container framework. Docker is a lightweight virtualization platform that has several properties such as versioning of file system operations, a modular design for distribution of software components as well as a sustainable community that make it attractive as a preservation tool. One community of collaborators at CERN is already exploring Docker to preserve high energy physics experiments. Our approach is to eventually provide a mechanism that captures the additional provenance of computational experiments in a machine readable approach using the W3C standard RDF data model that has been shown to aid in contextualization of scientific experiment.

By starting with a formal model of Docker and the provenance of Docker activities, we hope to provide a basis for 1) existing scientific workflow frameworks and their workflow descriptions to be captured within a Linux container environment 2) data to be integrated in a consistent manner and lastly 3) for a common description of the environment. The ultimate goal being to provide automated tools that “wrap” the existing Docker command line tool and infrastructure such that it is transparent to the scientist but captures information necessary to populate the metadata behind the scenes. Automated scientific gateways that utilize Docker as a deployment and execution platform would provide a further degree of transparency and allow researchers a low barrier for utilization.

Features:


## Brief Example of SmartContainers in action


## Docker



## RDF
The Resource Description Framework (RDF) is a framework for representing and linking data within the World Wide Web infrastructure [[RDF1.1](http://www.w3.org/TR/rdf11-concepts/)], and is a graph-based data model, where the core structure of the abstract syntax is a set of triples, each consisting of a subject, a predicate and an object.
![](https://github.com/charlesvardeman/sc_spec/blob/master/assets/rdf-graph.svg)

Solid uses several serialization syntaxes for storing and exchanging RDF such as [Turtle](http://www.w3.org/TR/turtle/) and [JSON-LD](http://www.w3.org/TR/json-ld/). When creating new RDF resources, the preferred **default** serialization is Turtle. Solid-compliant servers should implement content negotiation in order to handle different serialization formats.


## Linked Data Principles
Tim Berners-Lee published a [technical note](http://www.w3.org/DesignIssues/LinkedData.html) outlining some basic principles for publishing data on the Web.
## Ontologies

## Persistent Identifiers and minting URIs
CoolURIs, Trusty URI's WebID ORCID and RDA PID group work.
