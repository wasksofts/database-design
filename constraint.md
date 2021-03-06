## Updating check constraints in Postgres

This example demonstrates how to update check constraints in Postgres. Say if we have the following table with a check constraint:
```sql
CREATE TABLE post (
    id uuid DEFAULT gen_random_uuid() UNIQUE,
    type text CHECK (type = ANY (ARRAY['question'::text, 'answer'::text, 'comment'::text])),
    CONSTRAINT post_pkey PRIMARY KEY (id, type)
);
```

Find the name of the check constraint:
```sql
select pgc.conname as constraint_name,
       ccu.table_schema as table_schema,
       ccu.table_name,
       ccu.column_name,
       pgc.conkey as definition
from pg_constraint pgc
join pg_namespace nsp on nsp.oid = pgc.connamespace
join pg_class  cls on pgc.conrelid = cls.oid
left join information_schema.constraint_column_usage ccu
          on pgc.conname = ccu.constraint_name
          and nsp.nspname = ccu.constraint_schema
where contype ='c'
order by pgc.conname;
```
Output:
```
--post_type_check	public	post	type	{2}
```

There are no modify constraint in Postgres, so we need to drop the old constraint by name, and update the new one, preferably with the same name:

```sql
ALTER TABLE POST 
DROP CONSTRAINT post_type_check,
ADD CONSTRAINT post_type_check CHECK (type = ANY (ARRAY['question'::text, 'answer'::text, 'comment'::text]));
```


## Limitations

Using `check` constraints to check if the type is one of the values has the same limitation as Postgres's `enum` - we can only add values, we cannot remove an existing value. The example below demonstrates them by creating a table with two enum values `a` and `b` and inserting the values.
```sql
DROP TABLE pg_temp.hello;
CREATE TABLE IF NOT EXISTS pg_temp.hello (
	id int NOT NULL GENERATED ALWAYS AS IDENTITY,
	type text NOT NULL CHECK(type = ANY(ARRAY['a', 'b']))
);
```

If the inserted `type` is not `a` or `b`, an error would be thrown:
```sql
INSERT INTO hello (type) VALUES ('a'), ('b');
INSERT INTO hello (type) VALUES ('c');
--ERROR:  new row for relation "hello" violates check constraint "hello_type_check"
--DETAIL:  Failing row contains (3, c).
```

Once a row with `b` exists, we cannot remove the checking there:
```sql
ALTER TABLE pg_temp.hello
DROP CONSTRAINT hello_type_check,
ADD CONSTRAINT hello_type_check CHECK(type = ANY(ARRAY['a']));
--ERROR:  check constraint "hello_type_check" of relation "hello" is violated by some row
```

However, if we remove the row with type `b`, we can alter the constraint:
```sql
TRUNCATE TABLE pg_temp.hello;

ALTER TABLE pg_temp.hello
DROP CONSTRAINT hello_type_check,
ADD CONSTRAINT hello_type_check CHECK(type = ANY(ARRAY['a']));
```
