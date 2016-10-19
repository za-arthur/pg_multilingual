## Multilingual patch for PostgreSQL

Sometimes there is a need in text search using different dictionaries in same
text. In this case users combine necessary dictionaries into one dictionary
file. Usually it is hard task.

This patch allow to use different text search dictionaries in one text using one
text search configuration.

## Installation

This patch can be applyed to [master PostgreSQL](https://github.com/postgres/postgres):

    $ git clone git@github.com:select-artur/pg_multilingual.git
    $ git clone git@github.com:select-artur/postgres.git
    $ cd postgres
    $ git apply ../pg_multilingual/pg_multilingual_join.patch
    $ ./configure
    $ make
    $ make install

## Changes

### New option for text search configuration mapping: **JOIN**

In PostgreSQL's text search configuration for each token type you can specify a
list of dictionaries. PostgreSQL uses each dictionary to normalize words into
lexems using loop. Original PostgreSQL stops word normalizing if some dicitonary
recognizes it as a known word.

With option **JOIN** PostgreSQL return all recognized lexems until the end of a
list of dictionaries or a configuration mapping without this option are occured.
It is usefull if you don't know in what language a document is wrote.

## Usage

Let's suppose you already have **german_hunspell** and **english_hunspell**
dictionaries. You need to execute the following query to create multilingual
configuration:

```sql
=> CREATE TEXT SEARCH CONFIGURATION multi_conf (COPY=simple);
=> ALTER TEXT SEARCH CONFIGURATION multi_conf
	ALTER MAPPING FOR asciiword, asciihword, hword_asciipart,
	word, hword, hword_part
	WITH german_hunspell (JOIN), english_hunspell;
```

After this you can query documents with german and english words.

## Example

**apod_en_de.dump** is the dump of test base. To restore this dump you need
german and english hunspell dictionaries from [hunspell_dicts](https://github.com/postgrespro/hunspell_dicts):

    $ git clone git@github.com:postgrespro/hunspell_dicts.git
    $ cd hunspell_dicts
    $ make -C hunspell_en_us USE_PGXS=1 install
    $ make -C hunspell_de_de USE_PGXS=1 install

After this dictionary files were copied to the PostgreSQL share directory.

You need to restore the dump:

    $ psql apod < pg_multilingual/apod_en_de.dump

This command will create the following objects:

- **hunspell_en_us** and **hunspell_de_de** extensions
- **english_hunspell** and **german_hunspell** dictionaries
- **apod_conf** configuration
- **apod** table with english and german documents

Here example queries:

```sql
=> SELECT title FROM apod WHERE fts @@ to_tsquery('apod_conf', 'galaxy') LIMIT 1;
        title
---------------------
 The UV SMC from UIT
(1 row)

=> SELECT title FROM apod WHERE fts @@ to_tsquery('apod_conf', 'Galaxie') LIMIT 1;
             title
--------------------------------
 Andromeda - ein Inseluniversum
(1 row)
```
