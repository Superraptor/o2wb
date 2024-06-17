## Badges

[![License: CC0-1.0 Universal](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](http://creativecommons.org/publicdomain/zero/1.0/) [![Python 3.1](https://img.shields.io/badge/python-3.1-blue.svg)](https://www.python.org/downloads/release/python-300/)

<img src="o2wb.png" width="500" height="500">

# O2WB: A tool for ontology reuse in Wikibase

O2WB is a Python command line tool that enables ontology reuse in Wikibase, a popular data publishing platform. The tool aligns the Wikibase data model with RDF and facilitates the import and export of RDF Schema and OWL ontologies within Wikibase. This will help to tighten the connection between existing linked data resources and Wikibase instances.

This is a fork of the original GitHub repository in order to make it function for my specific purposes. To use this repository, clone it by running `git clone https://github.com/Superraptor/o2wb.git`. Installing using `pip` will install a different version. This README will not be relevant otherwise.


## Features

- Import ontologies from a local file or URL
- Follow owl:imports to recursively import ontologies 
- Represent IRIs, Blank Nodes, and Literals in the Wikibase data model
- Instantiate Statements that correspond to the Triples
- Correctly include Datatypes and Qualifiers
- Enable editing of imported ontologies through the Wikibase GUI
- Export ontologies in a variety of formats


## Requirements

- rdflib 6.2.0
- wikibase-integrator 0.12.3
- antlr4-python3-runtime 4.9.3
- argparse 1.4.0

## Installation

O2WB can be installed by cloning this repository to your machine and installing the required packages from requirements.txt (this has been updated in this fork and is different from the original repository).

Make sure the following requirements are fulfilled:
- [The full Wikibase Suite is installed.](#Set-up-a-Wikibase-Suite-instance)
- [A bot account with appropriate editing rights is created.](#Create-a-bot-account-in-Wikibase)

The next two sections guide through the process of fulfilling these requirements:
    
## Setting up a Wikibase Suite instance

The full Wikibase Suite required for O2WB to work consists of the MediaWiki base platform with Wikibase extensions, the MySQL (MariaDB) database and the Elasticsearch, QuickStatements and Wikidata query services (a Blazegraph SPARQL endpoint).

There are two main approaches to setting up a Wikibase Suite instance, a [Docker-based setup (extended install)](https://www.mediawiki.org/wiki/Wikibase/Docker) and a [full manual setup](https://www.mediawiki.org/wiki/Wikibase/Suite). Follow the corresponding links for the respective installation manuals. In the Docker-based setup, the components of the Wikibase Suite are automatically set up in Docker containers, making the installation and configuration process much easier. In the full manual setup, each component must be installed separately. We recommend using the Docker-based setup.

For this fork, it has only been tested in the Docker-based setup.


## Creating a bot account in Wikibase

In order to use O2WB, you need to set up a Wikibase bot with the necessary permissions. Follow these steps:

1. Go to `Special:SpecialPages` on your MediaWiki instance and click on `Bot passwords`.
2. Log in to your account.
3. Enter a `Bot name` into the section labeled `Create a new bot password`.
4. Click `Create`.
5. Select the following permissions: (1) high-volume (bot) access, (2) edit existing pages, and (3) create, edit, and move pages.
6. Make sure to get your bot's name (`username@botname`) and your bot password.

Now that you have set up the Wikibase bot account, you are (almost) ready. Next, update your `config.json` file with your MediaWiki instance information and your bot information.


## Importing an ontology

The process of importing an ontology into Wikibase is straightforward with the help of the o2wb-imp command or the manual import.py script.

To import an ontology, you need to have the following details:

- The location of the ontology to import, either as a file or a URL
- The name of the ontology

You can choose to import the ontology either non-recursively or recursively (by adding the --recursive flag). If you choose to import recursively, then all the dependencies of the ontology (owl:imports statements) will be imported along with the main ontology. You can also define the depth of the recursive imports (--recursive <depth>).

Additionally, you can create a csv mapping file that documents the mapping of the ontology IRIs to Wikibase Properties and Items (--mapping <Path to the mapping file>). 

To run the import process, open a terminal or command prompt and run the following command:

```bash
  python import.py {--url <URL of your ontology> | --file <path to ontology file>} --name <ontology name> [--recursive <Depth of recursive imports>] [--mapping <path to export the mapping scheme>]
```

Replace the values in the brackets with your own values and run the command. The import process will start, and you will see the progress of the import in the terminal.

Note that the import process can take a while, depending on the size of the ontology and the number of dependencies. You can monitor the progress of the import and then browse all newly created Properties and Items on the Recent Changes Special Page of the Wikibase UI.


## Editing an ontology

After having imported the ontology, we now can edit it using the Wikibase UI. There are three types of edits that we look at: 
- adding new Statements
- editing existing Statements
- deleting Statements

To add a new Statement, first consider the Entities that make it up as well as their respective Datatypes in Wikibase. 

&#x26a0;&#xfe0f; If such an Entity does not exist yet or there is no Property that has the appropriate Datatype to be used in the Statement, you should create it through either the Create a new Item or Create a new Property Special Page. When creating a new Entity, use the o2wb:representsIRI Property to indicate the IRI that will be exported later or the o2wb:typeOf Property with the value of o2wb:BNodeRepresentation to indicate that the Entity is a Blank Node.

After making sure that all the necessary Entities exist, navigate to the page of the Entity that is supposed to be the subject of the new Statement. On this page you can add a new Statement selecting the already existing Entities. 

&#x26a0;&#xfe0f; To include the recently created Statement in an ontology, add a Reference with the Property o2wb:fromOntology and an Item that stands for the respective ontology.

In a similar manner, you can edit Statements by editing or replacing their constituent Entities or chaning their non-Entity Values. 

When replacing an Entity, make sure that it is created beforehand and has an appropriate Datatype. Finally, you can exclude Statements from a given ontology by deleting them or removing their o2wb:fromOntology References.
## Exporting an ontology

The o2wb-exp command exports an ontology stored in your Wikibase instance to a file in RDF format. The --file option specifies the output file name in any of the common RDF serialization formats ([supported serializations](https://rdflib.readthedocs.io/en/stable/pluginserializers.html)) and the --name option specifies the name of the ontology you want to export.

Analogously to the o2wb-imp command, you can also use the --mapping option to create a mapping file that maps the IRIs in the exported RDF file to Iteams and Properties in Wikibase. Please note that the exported RDF file will contain only the data related to the specified ontology.

To run the import process, open a terminal or command prompt and run the following command:

```bash
  python export.py --file <path to ontology file> --name <ontology name> [--mapping <path to export the mapping scheme>]
```

Note that the export process can take a while depending on the size of the ontology. You can monitor the progress of the export with a progress bar. Exported RDF files might have a different presentation from the initial imported file: order of triples and defined prefixes might differ despite the exported files being identical with regards to their content.


## Examples

To import the DCAT ontology from a URL, run:

```bash
  python import.py --url "https://www.w3.org/TR/vocab-dcat-2/" --name "DCAT Ontology" 
```

To also recursively import one level of owl:imports ontologies, run:

```bash
  python import.py --url "https://www.w3.org/TR/vocab-dcat-2/" --name "DCAT Ontology" --recursive 1
```

To import the FOAF ontology from a file and create a mapping file, run:

```bash
  python import.py --file "pathTo/foaf.owl" --name "FOAF Ontology"  --mapping "pathTo/mapping.csv"
```

To export the FOAF ontology in Turtle format, run:

```bash
  python export.py --file "pathTo/foafExported.ttl" --name "FOAF Ontology" 
```

To also generate a mapping, run:

```bash
  python export.py --file "pathTo/foafExported.ttl" --name "FOAF Ontology"  --mapping "pathTo/mapping.csv"
```


## Contributing

Contributions are welcome. Please open an issue or submit a pull request if you would like to contribute.
## License

O2WB is licensed under the [CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) license.


## Authors
    
This is an anonymized version of the repository. We also offer a non-anonymized possibility to install the tool via PyPI for the final version.

## Acknowledgements

