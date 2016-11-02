# PostgreSQL homework

## Day 1

1. Select all the tables we created (and only those) from `pg_class`.

    ```
    SELECT relname FROM pg_class
    WHERE relname !~ '^(pg_|sql_)'
    AND relkind = 'r';
    ```

2. Write a query that finds the country name of the LARP Club event.

    ```
    SELECT c.country_name FROM countries c
    JOIN venues v ON c.country_code = v.country_code
    JOIN events e ON v.venue_id = e.venue_id
    WHERE e.title = 'LARP Club';
    ```

3. Alter the `venues` table to contain a boolean column called `active`, with the default value of `true`.

    ```
    ALTER TABLE venues
    ADD COLUMN active BOOLEAN DEFAULT TRUE;
    ```
