Daybed
======

... or spatial backend as a service !

* Mathieu Leplatre *(@leplatrem)*
* PostGIS and Leaflet enthusiast *(training, plugins, Django apps)*
* JavaScript and Python dev *(freshman at Mozilla)*

http://leplatrem.github.io/slides/201501-fosdem/

.. notes::

    Talk about background : IT services company, building spatial apps.

    Build small prototypes quickly to bait customers :)

    Almost always storing lat/lng with attributes.

----

How would you build... ?
========================

.. notes::

    Ask the audience about their strategy when they need to store spatial data,
    like tracking pictures of a specific flower.
    Ask what happens when another app with other specific fields comes up,
    like for bar terrasses.

----

Why Daybed ?
============

* Build collaborative apps *(basic, spatial, encrypted, ...)*
* Build a Google Forms alternative *(form → model, submission → record)*
* Do not reinvent the wheel *(JavaScript app without API dev)*
* Prototype Web applications *(Frontend-dev only)*

----

Daybed is...
============

* A Web API *(REST, Create Read Update Delete)*
* A “*reusable*” backend *(1 instance → N different applications)*
* A tool for rapid application building *(...or prototyping)*

----

Daybed is not...
================

* Another SGBD *(too naive)*
* A framework *(client agnostic)*
* The golden hammer *(not everything is a nail)*

----

Just a simple API !
===================

.. image:: showing-usage-of-daybed-in-one-picture

----

Under the hood
==============

* **Pyramid**, a robust and powerful python Web framework
* **Cornice**, a REST framework for Pyramid
* **Colander** for the schema validation part
* **Redis** as the default persistence backend *(or CouchDB)*
* **ElasticSearch** as indexing and faceted search engine *(pluggable)*

----

Model definition
================

* Model id
* List of fields *(int, string, relations, ...)*
* List of permissions

→ REST endpoints ``/models/<id>/definition``, ``/models/<id>/records``

----

Key features
============

* Record validation *(from model schema)*
* Authentication *(Hawk tokens)*
* Permissions *(CRUD, by author etc.)*

----

Spatial features
================

* Geometries field types *(point, lines, polygons, geojson...)*
* GeoJSON content type *(feature collection)*
* Spatial indexing *(bounding box, distance, ...)*

----

Permissions
===========

* Creator of model has full permissions

Optional matrix, by model:

* userid, anonymous, authenticated
* create, read (all|own), update (all|own) delete (all|own)
* (read|modify|delete) model definition

----

Daybed.js
=========

* Wrap HTTP requests
* Uses promises *(with polyfill)*
* Authentication tokens *(Hawk signing)*

----

Getting started
===============

.. code-block :: javascript

    var definition = {
      title: 'FOSDEM',
      description: 'Simple locations',
      fields : [
        {name: 'location', type: 'point'},
        {name: 'label', type: 'string'},
      ]
    };

    var permissions = {
      'Everyone': ['create_record', 'read_all_records',
                   'update_all_records', 'delete_all_records']
    };

* `All available field types <http://daybed.readthedocs.org/en/latest/fieldtypes.html>`_.
* `Permissions documentation <http://daybed.readthedocs.org/en/latest/permissions.html#models-permissions>`_.

----

Getting started
===============

.. code-block :: html

    <script src="//js.daybed.io/build/daybed.js"></script>


.. code-block :: javascript

    var server = 'https://daybed.io';
    var modelId = 'a-simple-location-model-with-label';
    var model = {
      definition: definition,
      permissions: permissions
    };

    Daybed.startSession(server)
      .then(function (session) {
        return session.saveModel(modelId, model);
      });

----

Load records
============

.. code-block :: javascript

    var session = new Daybed.Session(server);

    session.getRecords(modelId, {
        format: 'application/vnd.geo+json',
      })
      .then(function (geojson) {
        L.geoJson(geojson).addTo(map);
      });

----

Create records
==============

.. code-block :: javascript

    map.on('dblclick', function(e) {
      // LatLng to [x, y]
      var point = [e.latlng.lng, e.latlng.lat];
      session.saveRecord(modelId, {
          label: 'Building',
          location: point
        })
        .then(function(record) {
          var m = L.marker(e.latlng).addTo(map);
          m._recordId = record.id;
        });
    });

----

Modify and delete
=================

.. code-block :: javascript

    layer.on('click', function () {
      session.deleteRecord(model, layer._recordId)
        .then(function () {
          map.removeLayer(layer);
        });
    });

* RESTful verbs *(PUT, PATCH, DELETE)*
* ``session.deleteRecord(modelId, id)``, ``session.saveRecord(modelId, record)``

----

Share authentication token
==========================

* Shared token → collaborative app!

For example, use token from URL hash or create a new one if missing:

.. code-block :: javascript

    var token = window.location.hash.slice(1);

    Daybed.startSession(server, {token: token})
      .then(function (session) {
        window.location.hash = session.token;
      })
      .catch(function (e) {
        console.error("Could not start session", e);
      });

----

Lookup records
==============

* E/S mappings are generated from model definitions
* Records are indexed on creation
* Every basic geometric types
* Operators on BBox, distance
* Geo point aggregates *(a.k.a. clustering, via `plugin <https://github.com/zenobase/geocluster-facet>`_)*

The best Web companion !

* Sorts, paginates, aggregates, counts
* Scales; Insanely fast
* Ubiquitous

----

Bounding box search
===================

* Build queries in JSON !

.. code-block :: javascript

    var query = {
      ...
        filter: { geo_bounding_box : {
            location: {
              top: bbox.getNorthWest().lat, left: bbox.getNorthWest().lng,
              bottom: bbox.getSouthEast().lat, right: bbox.getSouthEast().lng
            }
          } }
      ...
    };

    session.searchRecords(modelId, query)
      .then(function (response) {
        alert(response.hits.hits.length + ' results!');
      });

* `ElasticSearch Query DSL <http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-geo-bounding-box-filter.html#query-dsl-geo-bounding-box-filter>`_

----

Generic API means...
====================

* No effort on backend *(quick start)*
* Logic-less API *(very basic rules)*
* More work on the client *(computation, conflicts)*
* Easier with schemaless database *(...or PostgreSQL json!)*

→ Daybed on server + `Turf.js <http://turfjs.org>`_ on client ?

----

Conclusion
==========

* Deploy one backend! Roll out many applications!

→ Think twice before implementing a custom backend!

`Github <https://github.com/spiral-project>`_ — https://daybed.io/v1/

See similar tools (`postgrest <https://github.com/begriffs/postgrest>`_, `Eve <http://python-eve.org>`_)

----

Next steps
==========

* `Form builder <https://github.com/spiral-project/formbuilder>`_
* A bit more HATEOAS
* Websockets / SimplePush
* Precondition headers
