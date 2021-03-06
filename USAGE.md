# Usage

First you need a working install of Graphite and Carbon: http://graphite.readthedocs.io/en/latest/install-virtualenv.html

Start by installing biggraphite:
```bash
$ # Last release version
$ pip install biggraphite
$ # From git
$ pip install -U https://github.com/criteo/biggraphite/archive/master.zip
```

Configure Cassandra (you will probably want to tweak the keyspace).

```bash
$ ${CASSANDRA_HOME}/bin/cqlsh < share/schema.cql
# Can be slow on HDD. Restart-it if it timeouts.
$ bgutil --cassandra_contact_points=localhost syncdb --storage-schemas storage-schemas.conf
# To create arbitrary retentions and see the schema:
$ bgutil --cassandra_contact_points=localhost syncdb --dry_run --retention "86400*1s:10080*60s" 
```

## Carbon plugin

In carbon.conf set the following settings:
```text
BG_CASSANDRA_KEYSPACE = biggraphite
BG_CASSANDRA_CONTACT_POINTS = 127.0.0.1
DATABASE = biggraphite
```

You can optionally use `whisper+biggraphite` or `biggraphite+whisper` to enable
double writes.

You'll need to start `bg-carbon-cache` instead of `carbon-cache` to run carbon with
biggraphite. If carbon is installed in `/opt/graphite/lib` and not in site-packages
you'll need to set do set `PYTHONPATH=/opt/graphite/lib`.

## Graphite Web plugin

In local_settings.py set the following settings:

```python
STORAGE_FINDERS = ['biggraphite.plugins.graphite.Finder']
TAGDB = 'biggraphite.plugins.tags.BigGraphiteTagDB'
BG_CASSANDRA_KEYSPACE = 'biggraphite'
BG_CASSANDRA_CONTACT_POINTS = '127.0.0.1'
```

You can optionally keep the StandardFinder to enable double reads from whisper
and biggraphite.