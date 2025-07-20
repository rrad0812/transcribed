
The client-server interaction
=============================
The client application sends a request to the server each time the following methods 
are called:
  
* load method of the task,
* open,
* apply,
* server,
* total_records methods of a data item or
* print method of a report item.

The total_records method is called by a table to display the number of pages in 
the paginator.

All these methods use the *process_request* method of the task to send a request 
to the server. You can set a breakpoint there to check all client-server interactions.
The *process_request* method sends to the server the following information:

* *name of the request*
* *ID of the task*
* *ID of the item*, whose method was called
* *request parameters*

.. note::
  Each item of the task tree has a unique ID attribute.

The server on receiving the request, based on passed IDs, finds the corresponding 
task (it can be Project task or Administrator task) and the item on the server, 
executes an appropriate method of the item with passed parameters and returns 
the result of the execution to the client.

The server methods can trigger events that can modify their default behavior.

Every item of the task tree on the server has the

* the *environ* and
* the *session*

attributes that store context of the current request.

Let's take a closer look at the *server* method of the item on the client. You can 
use it to execute a function declared in the server module of an item.For example, 
we can execute the test function declared in the server module of the catalog item:

.. code-block:: py
  
  def test(item, a, b):
    return a + b

by using the *server* method

.. code-block:: js

  task.catalog.server('test', [1,2], function(result, error) {
    console.log(result, error)
  })

We pass it, the name of the function and the list of parameters.

.. note::
	The first parameter of the function on the server must be an item object.

If a callback function is specified, the function on the server is executed asynchronously, 
after which the callback is executed with parameters that are:

* the *result* of the server function and 
* the *error text*, if an exception was raised during the server function execution.

Otherwise the server method is executed synchronously and returns the result of 
execution. If an exception was raised during execution, the server method throws 
an exception.

Let's raise an Exception:

.. code-block:: py

  def test(item, a, b):
    raise Exception('Some error')
    return a + b

.. code-block:: js

  task.catalog.server('test', [1,2], function(result, error) {
    console.log(result, error)
  })

Knowing all of these you can access the *environ* and *session* attributes and call 
the *can_view*, *can_create*, *can_edit*, *can_delete* methods.

.. code-block:: py
  
  def test(item, a, b):
    print 'Session:', item.session
    print 'Environment', item.environ
    return a + b


.. code-block:: js

  task.catalog.server('test', [1,2], function(result, error) {
    console.log(result, error)
  })

Let's set a safe mode and change the privileges of the user role

.. code-block:: py

  def test(item, a, b):
    print 'User_name', item.session['user_info']['user_name']
    print 'Role_id', item.session['user_info']['role_id']
    print 'Can view', item.can_view()
    print 'Can create', item.can_create()
    print 'Can edit', item.can_edit()
    print 'Can delete', item.can_delete()
    return a + b

You can use *item.session['user_info']['role_id']* or item's *can_view*,
*can_create*, *can_edit*, *can_delete* methods to limit user rights.

.. code-block:: py

  def test(item, a, b):
    if item.session['user_info']['role_id'] == 2:
        raise Exception('%s, you are not allowed to perform this operation' % \
            item.session['user_info']['user_name'])
    return a + b

.. code-block:: js

  task.catalog.server('test', [1,2], function(result, error) {
    if (error) {
      task.warning(error)
    }
    else {
      console.log(result)
    }
  })

Besides this client-server exchange mechanism you can download files from server 
static folder, upload files to the server, using the client task upload method,
and process post requests from other application or services by using the *on_ext_request*
event handler by the server task.
