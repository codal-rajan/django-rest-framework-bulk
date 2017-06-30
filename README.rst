Django REST Framework Bulk
==========================

.. image:: https://badge.fury.io/py/djangorestframework-bulk.png
    :target: http://badge.fury.io/py/djangorestframework-bulk

.. image:: https://travis-ci.org/miki725/django-rest-framework-bulk.svg?branch=master
    :target: https://travis-ci.org/miki725/django-rest-framework-bulk

Django REST Framework bulk CRUD view mixins.

Overview
--------

Django REST Framework comes with many generic views however none
of them allow to do bulk operations such as create, update and delete.
To keep the core of Django REST Framework simple, its maintainer
suggested to create a separate project to allow for bulk operations
within the framework. That is the purpose of this project.

Requirements
------------

* Python>=2.7
* Django>=1.3
* Django REST Framework >= 3.0.0
* REST Framework >= 2.2.5
  (**only with** Django<1.8 since DRF<3 does not support Django1.8)

Installing
----------

Using pip::

    $ pip install djangorestframework-bulk

or from source code::

    $ pip install -e git+http://github.com/miki725/django-rest-framework-bulk#egg=djangorestframework-bulk

Example
-------

The bulk views (and mixins) are very similar to Django REST Framework's own
generic views (and mixins)::

    
    from rest_framework_bulk.drf3.serializers import (
        BulkListSerializer,
        BulkSerializerMixin,
    )

    from rest_framework_bulk.generics import ListBulkCreateUpdateDestroyAPIView
    
    # **serializer.py**

    class FooSerializer(BulkSerializerMixin, ModelSerializer):
        class Meta(object):
            model = FooModel
            # only necessary in DRF3
            fields = '__all__'
            list_serializer_class = BulkListSerializer

    # **views.py**

    class FooViewSet(ListBulkCreateUpdateDestroyAPIView):
        queryset = FooModel.objects.all()
        serializer_class = FooSerializer


    # **urls.py**

    from rest_framework import routers
    from .views import UserViewSet , FooViewSet
    from rest_framework_bulk.routes import BulkRouter
    
    
    router = routers.DefaultRouter()
    router.register(r'users', UserViewSet)
    
    router2 = BulkRouter()
    router2.register(r'foo', FooViewSet)

    urlpatterns = []
    urlpatterns += router.urls
    urlpatterns += router2.urls

The above will allow to create the following queries

::

    # list queryset
    GET

::

    # create single resource
    POST
    {"field":"value","field2":"value2"}     <- json object in request data

::

    # create multiple resources
    POST
    [{"field":"value","field2":"value2"}]

::

    # update multiple resources (requires all fields)
    PUT
    [{"field":"value","field2":"value2"}]   <- json list of objects in data

::

    # partial update multiple resources
    PATCH
    [{"field":"value"}]                     <- json list of objects in data

::

    # delete queryset (see notes)
    DELETE

Router
------

The bulk router can automatically map the bulk actions::

    from rest_framework_bulk.routes import BulkRouter

    class UserViewSet(BulkModelViewSet):
        model = User

        def allow_bulk_destroy(self, qs, filtered):
            """Don't forget to fine-grain this method"""

    router = BulkRouter()
    router.register(r'users', UserViewSet)
