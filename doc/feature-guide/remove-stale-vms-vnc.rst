.. This work is licensed under the Creative Commons Attribution 4.0 International License.
   To view a copy of this license, visit http://creativecommons.org/licenses/by/4.0/ or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.

==============================================================
Removing Stale Virtual Machines and Virtual Machine Interfaces
==============================================================

This topic gives examples for removing stale VMs (virtual machines) and VMIs (virtual machine interfaces). Before you can remove a stale VM or VMI, you must first remove any back references associated to the VM or VMI.

-  `Problem Example`_ 


-  `Show Virtual Machines`_ 


-  `Show Virtual Machines Using Python API`_ 


-  `Delete Methods`_ 



Problem Example
===============

The troubleshooting examples in this topic are based on the following problem example. A ``net-delete`` of the virtual machine 2a8120ec-bd18-49f4-aca0-acfc6e8fe74f returned the following messages that there are two VMIs that still have back-references to the stale VM.
The two VMIs must be deleted first, then the Neutron ``net-delete<vm_ID>`` command will complete without errors.

::

 From neutron.log:

 2014-03-10 14:18:05.208    

 DEBUG [urllib3.connectionpool]

 "DELETE/virtual-network/2a8120ec-bd18-49f4-aca0-acfc6e8fe74f HTTP/1.1" 409 203

 2014-03-10 14:18:05.278    

 ERROR [neutron.api.v2.resource] delete failed

 Traceback (most recent call last):

   File "/usr/lib/python2.7/dist-packages/neutron/api/v2/resource.py", line

 84, in resource

     result = method(request=request, **args)

   File "/usr/lib/python2.7/dist-packages/neutron/api/v2/base.py", line

 432, in delete

     obj_deleter(request.context, id, **kwargs)

   File

 "/usr/lib/python2.7/dist-packages/neutron/plugins/juniper/contrail/contrail

 plugin.py", line 294, in delete_network

     raise e

 RefsExistError: Back-References from

 http: //127.0.0.1:8082/virtual-machine-interface/51daf6f4-7366-4463-a819-bd1

 17fe3a8c8,

 http: //127.0.0.1:8082/virtual-machine-interface/30882e66-e175-4fbb-862e-354

 bb700b579 still exist 


Show Virtual Machines
=====================

Use the following command to show all of the virtual machines known to the Contrail API server. Replace the variable `` *<config-node-IP>* `` shown in the example with the IP address of the ``config-node`` in your setup.

``http://<config-node-IP>:8082/virtual-machines`` 

Example
-------

In the following example, 03443891-99cc-4784-89bb-9d1e045f8aa6 is a stale VM that needs to be removed.
  
::

 virtual-machines:

 	[

 		{

 			href:"http: //example-node:8082/virtual-machine/03443891-99cc-4784-89bb-9d1e045f8aa6",

 			fq_name:

 				[

 				"03443891-99cc-4784-89bb-9d1e045f8aa6"

 				],

 			uuid:"03443891-99cc-4784-89bb-9d1e045f8aa6"

 		},

 When the user attempts to delete the stale VM, a message displays that children to the VM still exist:


::

 root@example-node:~# curl -X DELETE -H "Content-Type: application/json; charset=UTF-8" http: //127.0.0.1:8082/virtual-machine/03443891-99cc-4784-89bb-9d1e045f8aa6   
 Children http: //127.0.0.1:8082/virtual-machine-interface/0c32a82a-7bd3-46c7-b262-6d85b9911a0d still exist  
 root@example-node:~#  

The user opens http: //example-node:8082/virtual-machine/ 03443891-99cc-4784-89bb-9d1e045f8aa6, and sees a ``virtual-machine-interface`` (VMI) attached to it. The VMI must be removed before the VM can be removed.
However, when the user attempts to delete the VMI from the stale VM, they get a message that there is still a back-reference:

::

 root@example-node:~# curl -X DELETE -H "Content-Type: application/json; charset=UTF-8" http: //<example-IP>:8082/virtual-machine-interface/0c32a82a-7bd3-46c7-b262-6d85b9911a0d

 Back-References from http: //<example-IP>:8082/instance-ip/6ffa29a1-023f-462b-b205-353da8e3a2a4 still exist

 root@example-node:~# 

Because there is a back-reference from an ``instance-ip`` object still present, the ``instance-ip`` object must first be deleted, as follows:

::

 root@example-node:~# curl -X DELETE -H "Content-Type: application/json; charset=UTF-8" http: //<example-IP>:8082/instance-ip/6ffa29a1-023f-462b-b205-353da8e3a2a4

 root@example-node:~# 

When the ``instance-ip`` is deleted, then the VMI and the VM can be deleted.


.. note:: To prevent inconsistency, be certain that the VM is not present in the Nova database before deleting the VM.




Show Virtual Machines Using Python API
======================================

The following example shows how to view virtual machines using a Python API. This example shows virtual machines and back-references. Once you identify back-references and existing children, you can delete them first, then delete the stale VM.

::

 root@example-node:~# source /opt/contrail/api-venv/bin/activate

 File "<stdin>", line 1, in <module>

   File "/opt/contrail/api-venv/lib/python2.7/site-packages/vnc_api/gen/vnc_api_client_gen.py", line 3793, in virtual_machine_interface_delete

     content = self._request_server(rest.OP_DELETE, uri)

   File "/opt/contrail/api-venv/lib/python2.7/site-packages/vnc_api/vnc_api.py", line 342, in _request_server

     raise RefsExistError(content)

 cfgm_common.exceptions.RefsExistError: Back-References from http: // <example-IP>:8082/instance-ip/6ffa29a1-023f-462b-b205-353da8e3a2a4 still exist

 >>> (api-venv)root@example-node:~# python

 Python 2.7.5 (default, Mar 10 2014, 03:55:35) 

 [GCC 4.6.3] on linux2

 Type "help", "copyright", "credits" or "license" for more information.

 >>> from vnc_api.vnc_api import VncApi

 >>> vh=VncApi()

 >>> vh.virtual_machine_interface_delete(id='0c32a82a-7bd3-46c7-b262-6d85b9911a0d')


Traceback (most recent call last):

::

 File "<stdin>", line 1, in <module>

   File "/opt/contrail/api-venv/lib/python2.7/site-packages/vnc_api/gen/vnc_api_client_gen.py", line 3793, in virtual_machine_interface_delete

     content = self._request_server(rest.OP_DELETE, uri)

   File "/opt/contrail/api-venv/lib/python2.7/site-packages/vnc_api/vnc_api.py", line 342, in _request_server

     raise RefsExistError(content)

 cfgm_common.exceptions.RefsExistError: Back-References from http: // <example-IP>:8082/instance-ip/6ffa29a1-023f-462b-b205-353da8e3a2a4 still exist

 >>> 


Delete Methods
==============

Use help ( ``vh`` ) to show all delete methods supported.

Typical commands for deleting VMs and VMIs include:

-  ``virtual_machine_delete()`` to delete a virtual machine


-  ``instance_ip_delete()`` to delete an ``instance-ip`` .


