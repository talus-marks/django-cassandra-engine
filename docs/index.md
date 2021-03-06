# django-cassandra-engine docs

## Overview

*django-cassandra-engine* is a database wrapper for *Django Framework*.
It uses latest *Cqlengine*, which is currently the best Cassandra CQL 3 Object Mapper for Python.

---

## Features

* working `syncdb`, `migrate`, `sync_cassandra` and `flush` commands
* support for creating/destroying test database
* accepts all `Cqlengine` and `cassandra.cluster.Cluster` connection options
* automatic connection/disconnection handling
* support for multiple databases (also relational)

---

## Requirements

* Cassandra (of course)
* cqlengine
* django-nonrel
* djangotoolbox
* Django (1.6 or 1.7)
* blist (optional)

---

## Download

[Github](https://github.com/r4fek/django-cassandra-engine)

[PyPi](https://pypi.python.org/pypi/django-cassandra-engine)

---

## Installation

You can install it easily from PyPi by single command:

    pip install django-cassandra-engine

or clone source code and run:

    python setup.py install

---

## Getting started

1.  Add django-cassandra-engine to `INSTALLED_APPS` in your settings.py file:

        INSTALLED_APPS = ('django_cassandra_engine',) + INSTALLED_APPS


`IMPORTANT`: This app should be **the first app** on `INSTALLED_APPS` list.
This rule applies only to Django >= 1.7.

2.  Change `DATABASES` setting:

        DATABASES = {
            'default': {
                'ENGINE': 'django_cassandra_engine',
                'NAME': 'db',
                'TEST_NAME': 'test_db',
                'HOST': 'db1.example.com,db2.example.com',
                'OPTIONS': {
                    'replication': {
                        'strategy_class': 'SimpleStrategy',
                        'replication_factor': 1
                    }
                }
            }
        }

3.  Define some model:

        #  myapp/models.py
        import uuid
        from cqlengine import columns
        from cqlengine.models import Model

        class ExampleModel(Model):
            read_repair_chance = 0.05 # optional - defaults to 0.1
            example_id      = columns.UUID(primary_key=True, default=uuid.uuid4)
            example_type    = columns.Integer(index=True)
            created_at      = columns.DateTime()
            description     = columns.Text(required=False)

4.  Run `./manage.py sync_cassandra` in order to sync your models with Cassandra.

5.  Thats all!

---

## Advanced usage

Sometimes you want to use cassandra database along with your RDMS.
This is also possible! Just define your `DATABASES` like here:

    from cassandra import ConsistencyLevel

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        },
        'cassandra': {
            'ENGINE': 'django_cassandra_engine',
            'NAME': 'db',
            'USER': 'user',
            'PASSWORD': 'pass',
            'TEST_NAME': 'test_db',
            'HOST': '127.0.0.1',
            'OPTIONS': {
                'replication': {
                    'strategy_class': 'SimpleStrategy',
                    'replication_factor': 1
                },
                'connection': {
                    'consistency': ConsistencyLevel.ONE,
                    'lazy_connect': True,
                    'retry_connect': True
                    # + All connection options for cassandra.cluster.Cluster()
                },
                'session': {
                    'default_timeout': 10,
                    'default_fetch_size': 10000
                    # + All options for cassandra.cluster.Session()
                }
            }
        }
    }

Then run `./manage.py syncdb` for your regular database and
`./manage.py sync_cassandra` or `./manage.py syncdb --database cassandra`
for Cassandra DB.

All `cassandra.cluster.Cluster` and `cassandra.cluster.Session` options are well described
<a href="http://datastax.github.io/python-driver/api/cassandra/cluster.html" target="_blank" rel="nofollow">
    here
</a>.

---

## Using internal authorization

If you want to use
<a href="http://www.datastax.com/documentation/cassandra/2.1/cassandra/security/secure_config_native_authorize_t.html" target="_blank" rel="nofollow">
    internal authorization
</a>
just provide `USER` and `PASSWORD` in cassandra's database alias.

    ...
    'cassandra' {
        'ENGINE': 'django_cassandra_engine',
        'NAME': 'db',
        'USER': 'user',
        'PASSWORD': 'pass'
    }

You can also pass custom
<a href="http://datastax.github.io/python-driver/api/cassandra/auth.html#cassandra.auth.PlainTextAuthProvider" target="_blank" rel="nofollow">
    auth_provider
</a>
to `connection` dict:

    ...
    'connection': {
        'consistency': ConsistencyLevel.ONE,
        'lazy_connect': True,
        'retry_connect': True,
        'port': 9042,
        'auth_provider': PlainTextAuthProvider(username='user', password='password')
        # + All connection options for cassandra.cluster.Cluster()
    }

---

## Working with source code and running tests

    git clone https://github.com/r4fek/django-cassandra-engine.git
    cd django_cassandra_engine
    # mkvirtualenv cassengine
    pip install -r requirements.txt
    pip install Django#==1.7/1.6
    python setup.py install && ./testproject/runtests.py


## Contributing

You are highly encouraged to participate in the development, 
simply use GitHub's fork/pull request system.
