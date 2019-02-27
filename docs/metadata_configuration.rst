Panhandler Metadata Files
=========================

The heart of Panhandler is the `.meta-cnc.yaml` file. This allows a set of configuration snippets to be shared and
consumed as a single unit. For example, to configure a default security profile you may need to configure multiple
different parts of the PAN-OS configuration. Panhandler allows you to group those different 'pieces' and share them
among different devices as a single unit. Often times these configuration bits (affectionately called 'skillets')
need slight customization before deployment to a new device. The `.meta-cnc.yaml` file provides a means to templatize
these configurations and present a list of customization points, or variables, to the end user or consumer.

Basic concepts
--------------

In order to add multiple 'bits' of configuration to a device, we need to know the following things:

* XML Configuration fragment with optional variables defined in jinja2 format
* xpath where this xml fragment should be inserted into the candidate configuration
* the order in which these XML fragments must be inserted
* a list of all variables that require user input
* target version requirements. For example: PAN-OS 8.0 or higher

This is all accomplished by adding multiple files each containing an XML configuration fragment and a `.meta-cnc.yaml`
file that describes the load order, variables, target requirements, etc.


YAML syntax
-----------

Each `skillet` is structured as a series of files in a single directory. This directory may contain
a number of template files (XML, YAML, JSON, etc) and a `.meta-cnc.yaml` file. Note the following:

1. A `.meta-cnc.yaml` file that is formatted with using YAML with the following format:

.. code-block:: yaml

    name: config_set_name
    description: config_set description
    extends: gsb_baseline
    target_version: 8.1+
    variables:
      - name: INF_NAME
        description: Interface Name
        default: Ethernet1/1
        type_hint: text
    snippets:
      - xpath: some/xpath/value/here
        name: config_set_knickname
        file: filename of xml snippet to load that should exist in this directory


2. Multiple configuration files. Each should contain a valid template fragment and may use jinja2 variables.
   These templates may be XML, JSON, YAML, Text, etc. For PAN-OS devices, these are XML fragments from specific stanzas
   of the PAN-OS device configuration tree.


Snippet details
---------------

Each .meta-cnc.yaml file must contain the following top-level keys:

* name: name of this configuration set
* description: Short description
* extends: name of another skillet that is a dependency of this one
* target_version: String referring to target version requirements. I.E This skillet applies only to PAN-OS 8.1 or higher
* variables: Described in detail below
* snippets: a dict containing the following keys

    * name: knickname of the config_set
    * file: relative path to the configuration template
    * xpath (optional): XPath where this fragment belongs in the target OS hierarchy (for XML config_sets)

Each config_set can define nulitple variables that will be interpolated using the Jinja2 templating language. Each
variable defined in the `variables` list should define the following:


1. name: The name of the variable found in the config_sets. For example:

.. code-block:: jinja

    {{ name }}


2. description: A brief description of the variable and it's purpose in the configuration
3. label: Human friendly label to display to user
4. extends: Name of another config_set to load
5. default: A valid default value which will be used if not value is provided by the user
6. type_hint: Used to constrain the types of values accepted. May be implemented by additional third party tools.
   Examples are `text`, `text_field`, `ip_address`, `password`, `dropdown`, and `checkbox`.


Hints
-----

Ensuring all variables are defined
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When working with a large amount of configuration temlates, it's easy to miss a variable definition. Use this one-liner
to find them all.

cd into a config_set dir and run this to find all vars

.. code-block:: bash

    grep -r '{{' . |  cut -d'{' -f3 | awk '{ print $1 }' | sort -u


YAML Syntax
^^^^^^^^^^^

YAML is notoriously finicky about whitespace and formatting. While it's a relatively simple structure and easy to learn,
it can often also be frustrating to work with, especially for large files. A good reference to use to check your
YAML syntax is the `YAML Lint site <http://www.yamllint.com/>`_.
