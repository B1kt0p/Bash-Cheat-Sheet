# Jinja2 Cheat Sheet

 A cheat sheet for Jinja2

## Types of syntax

```bash
{# this is comment #}     - comment
{{ name }}                - varible
{% name %}                - control syntax
```

## Template
! from jinja2 import Template
render - generates a document based on the loaded template (substituting the query into the template)  
The dictionary name keys and functions must match the function names in the template  
The dictionary name keys and functions must match the function names in the template  

```python
rom jinja2 import Template
rt01 = {
    "hostname": "rt-01-dc",
    "mgmt_ip": "10.255.255.32",
}
template = """
hostname {{ hostname }}
!
interface Loopback0
  description mgmt
  ip address {{ mgmt_ip }} 255.255.255.255
!
"""
tmpl = Template(template)
print(tmpl.render(rt01))               
```
## Environment
Environment - the main class for describing the environment

###  Loader - template loader
• FileSystemLoader(directory) - loader from the file system, the argument is the directory containing the templates.
env.get_template(template_filename)  
• DictLoader(dict) - loads from a dictionary. The keys are the template names, the values ​​are the templates themselves.
env.get_template("template_name")  
• FunctionLoader(func_name) - loads by calling a function that returns a
string (or None)  
env.get_template("func_arg")  
• ChoiceLoader([loader1, loader2]) - combines several loaders and tries them one by one  
```python
loader1 = FileSystemLoader("./templates1")
loader2 = FileSystemLoader("./templates2")
loader = ChoiceLoader([loader1, loader2])
```
###  Example FileSystemLoader(FileSystemLoader)
  ```python
import yaml
from jinja2 import Environment, FileSystemLoader, Template
templ_file = "02.simple.j2"
vars_file = "02.simple.yaml"
with open(vars_file, "r") as _file:
    rt01 = yaml.safe_load(_file)
env = Environment(loader=FileSystemLoader("."))
tmpl01 = env.get_template(templ_file) print(tmpl01.render(rt01))
with open(templ_file, "r") as _file:
    tmpl02 = Template(_file.read())
print(tmpl02.render(rt01))
```
###  Example FileSystemLoader(DictLoader)
  ```python
import yaml
from jinja2 import DictLoader, Environment
rt_file = "03.router.j2"
sw_file = "03.switch.j2"
vars_file = "03.vars.yaml"
with open(vars_file, "r") as _file:
    variables = yaml.safe_load(_file)
with open(rt_file, "r") as _file:
    rt_template = _file.read()
with open(sw_file, "r") as _file:
    sw_template = _file.read()
env = Environment(loader=DictLoader({"router": rt_template, "switch": sw_template}))
for device in variables:
config = env.get_template(device["type"]).render(device) with open(f"03.result.{device['name']}.txt", "w") as _file:
        _file.write(config)
```
### Jinja2. Variables

```python
{{ name }} - variable access
• {{ list[0] }} - list element access
• {{ dict.key }} - dictionary element access
• {{ dict['key'] }} - dictionary element access by key
```

### Jinja2. For Loop

• Defined in the {% ... %} statement block   
• Starts with {% for ... %} and ends with {% endfor %}   
Can iterate over both lists and dictionary statements   

### JJinja2. If Condition

• Defined in the {% ... %} statement block    
• Starts with {% if ... %} and ends with {% endif %} • {% elif ... %} and {% else %} for alternative branches   

## Jinja2. Whitespace Control

• trim_blocks - removes the first line break after a block
• lstrip_blocks - removes spaces and tabs from the beginning of a line before a block

```python  
env = Environment( loader=FileSystemLoader(“."), trim_blocks=True, lstrip_blocks=True,
)
template = Template(
    _file.read(),
trim_blocks=True,
lstrip_blocks=True, )
```

{%- - removes spaces before the block (including line breaks)   
-%} - removes spaces after the block (including line breaks)    
{%+ - disables lstrip_blocks before the block (lstrip_block=False)    

```python 
interface {{ name }}\n
··{% if action == "override" %}\n
··switchport trunk allowed vlan {{ vlan_id }}\n
··{% else %}\n
··switchport trunk allowed vlan add {{ vlan_id }}\n
··{% endif %}\n
!\n
```

## Jinja2. Filters
• Used to modify and process variable values ​​• Can be used consecutively
• Can take parameters

```python
{{ name | default("unknown") }} - default value (frequent filter - alias d)
{{ trunk_data.vlans | join(",") }} - concatenates list elements into a string (like join in Python)
{{ trunks | dictsort }} - sorts a dictionary
{{ loud | capitalize }} - works with string case, similar to lower, title, upper
{{ list | first }} - returns the first element of a list (last - last)
{{ "rm_%s_to_%s_%s" | format("PE", "RR", "MSK") }} - string formatting
{{ vlans | length }} - list length
{{ string_with_spaces | trim }} - removes spaces from the beginning and end of a string
{{ loud | lower | list | union(" ") }} - filter pipeline
```
### You can write it yourself
```python
from jinja2 import Environment, FileSystemLoader
from netaddr import IPNetwork
def shift_ip_address(addr, offest): return IPNetwork(addr).ip + offest
env = Environment(loader=FileSystemLoader(".")) env.filters["shift_ip_address"] = shift_ip_address
```
```python
{{ "192.168.0.0/23" | shift_ip_address(2) }} => 192.168.0.2
{{ "192.168.0.10/23" | shift_ip_address(22) }} => 192.168.0.32 
```
### Useful module
```bash
pip install j2ipaddr
```

## Jinja2. Tests
```python
{% if bgp_as is defined %} - whether the variable is defined or not
{% if flag is boolean %} - whether the variable is boolean or not
{% if data.vlans is iterable %} - iterable variable
{% elif data.vlans is number %} - or a number
```
Jinja2. Variable Definitions (Set)
```python
{% for interface, data in interfaces.items() %}
interface {{ interface }}
{% for ip in data.ipv4 %}
{% set addr = ip | ip_address %}
{% set mask = ip | ip_netmask %}
ip address {{ addr }} {{ mask }}
{% endfor %}
!
{% endfor %}
```

## Jinja2. Splitting Templates (include)
• The include directive allows you to add one template to another {% include ... %}   
• The variable file must be shared (contain variables for all templates)   
```python
{% for interface, data in interfaces.items() %}
interface {{ interface }}
{% for ip in data.ipv4 %}
ip address {{ ip | ip_address }} {{ ip | ip_netmask }}
{% endfor %}
!
{% endfor %}
```

```python
{% include "14.include.ospf.j2" %}
{% include "14.include.interfaces.j2" %}

router ospf 1
{% for interface, data in interfaces.items() %}
{% if data.ospf and data.ospf.status %}
{% for ip in data.ipv4 %}
network {{ ip | ip_network }} {{ ip | ip_wildcard }} area {{ data.ospf.area }}
{% endfor %}
{% endif %}
{% endfor %}
!
```
## Jinja2. Template Inheritance

• Base template - a blank (template). Contains everything that a regular template contains plus special sections {% block ... %} ... {% endblock %}   
• Child template - generated based on the base template, extending its contents via the {% extends ... %} directive   
• In a child template, all descriptions are contained within {% block vrf %} blocks; anything outside these blocks will be ignored.
• Child templates can override the contents of the base template blocks.   

```python
{% block services %}
service tcp-keepalives-in
service tcp-keepalives-out
service password-encryption
{% endblock %}
!
no ip domain lookup
ip ssh version 2
!
{% block ospf %}
router ospf 1
auto-cost reference-bandwidth {{ ospf_bw | default(200000) }}
{% endblock %}
!
{% block bgp %}
{% endblock %}
!
{% block interfaces %}
{% endblock %}
!
line con 0 
privilege level 15 
logging synchronous 
login authentication CON 
exec prompt timestamp 
stopbits 1
line vty 0 15 
logging synchronous 
exec prompt timestamp 
transport input all
!
```
```python
{% extends "15.extend.base.j2" %}
{% block services %}
! this will erase service block from base
{% endblock %}
{% block ospf %}
! my ospf configuration
!
{{ super() }} 
{% for networks in ospf %} 
network {{ networks.network }} area {{ networks.area }} 
{% endfor %}
{% endblock %}
----
this line will be skipped
----
{% block interfaces %}
```
! interface configuration
!
{% endblock %}
