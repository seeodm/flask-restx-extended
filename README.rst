====================
Flask RESTX Extended
====================

.. image:: https://github.com/python-restx/flask-restx/workflows/Tests/badge.svg?branch=master&event=push
    :target: https://github.com/python-restx/flask-restx/actions?query=workflow%3ATests
    :alt: Tests status
.. image:: https://codecov.io/gh/python-restx/flask-restx/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/python-restx/flask-restx
    :alt: Code coverage
.. image:: https://readthedocs.org/projects/flask-restx/badge/?version=latest
    :target: https://flask-restx.readthedocs.io/en/latest/
    :alt: Documentation status
.. image:: https://img.shields.io/pypi/l/flask-restx.svg
    :target: https://pypi.org/project/flask-restx
    :alt: License
.. image:: https://img.shields.io/pypi/pyversions/flask-restx.svg
    :target: https://pypi.org/project/flask-restx
    :alt: Supported Python versions
.. image:: https://badges.gitter.im/Join%20Chat.svg
    :target: https://gitter.im/python-restx?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge
    :alt: Join the chat at https://gitter.im/python-restx
.. image:: https://img.shields.io/badge/code%20style-black-000000.svg
    :target: https://github.com/psf/black
    :alt: Code style: black


Flask-RESTX-Extended is a fork of `Flask-RESTX <https://github.com/python-restx/flask-restx>`_.


This version fix reverse proxy swagger ui load error on Flask-RESTX.


Compatibility
=============

Flask-RESTX-Extended requires Python 2.7 or 3.4+.


Installation
============

You can install Flask-RESTX-Extended with pip:

.. code-block:: console

    $ pip install flask-restx-extended


Quick start
===========

With Flask-RESTX, you only import the api instance to route and document your endpoints.

.. code-block:: python

    from flask import Flask
    from flask_restx import Api, Resource, fields

    app = Flask(__name__)
    app.config["SWAGGER_BASEURL"] = "/base"
    api = Api(app, version='1.0', title='TodoMVC API',
        description='A simple TodoMVC API',
    )

    ns = api.namespace('todos', description='TODO operations')

    todo = api.model('Todo', {
        'id': fields.Integer(readonly=True, description='The task unique identifier'),
        'task': fields.String(required=True, description='The task details')
    })


    class TodoDAO(object):
        def __init__(self):
            self.counter = 0
            self.todos = []

        def get(self, id):
            for todo in self.todos:
                if todo['id'] == id:
                    return todo
            api.abort(404, "Todo {} doesn't exist".format(id))

        def create(self, data):
            todo = data
            todo['id'] = self.counter = self.counter + 1
            self.todos.append(todo)
            return todo

        def update(self, id, data):
            todo = self.get(id)
            todo.update(data)
            return todo

        def delete(self, id):
            todo = self.get(id)
            self.todos.remove(todo)


    DAO = TodoDAO()
    DAO.create({'task': 'Build an API'})
    DAO.create({'task': '?????'})
    DAO.create({'task': 'profit!'})


    @ns.route('/')
    class TodoList(Resource):
        '''Shows a list of all todos, and lets you POST to add new tasks'''
        @ns.doc('list_todos')
        @ns.marshal_list_with(todo)
        def get(self):
            '''List all tasks'''
            return DAO.todos

        @ns.doc('create_todo')
        @ns.expect(todo)
        @ns.marshal_with(todo, code=201)
        def post(self):
            '''Create a new task'''
            return DAO.create(api.payload), 201


    @ns.route('/<int:id>')
    @ns.response(404, 'Todo not found')
    @ns.param('id', 'The task identifier')
    class Todo(Resource):
        '''Show a single todo item and lets you delete them'''
        @ns.doc('get_todo')
        @ns.marshal_with(todo)
        def get(self, id):
            '''Fetch a given resource'''
            return DAO.get(id)

        @ns.doc('delete_todo')
        @ns.response(204, 'Todo deleted')
        def delete(self, id):
            '''Delete a task given its identifier'''
            DAO.delete(id)
            return '', 204

        @ns.expect(todo)
        @ns.marshal_with(todo)
        def put(self, id):
            '''Update a task given its identifier'''
            return DAO.update(id, api.payload)


    if __name__ == '__main__':
        app.run(debug=True)
