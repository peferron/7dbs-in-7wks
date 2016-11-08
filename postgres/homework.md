# PostgreSQL homework

## Day 1


### 1.

```
SELECT relname FROM pg_class
WHERE relname !~ '^(pg_|sql_)'
AND relkind = 'r';
```

### 2.

```
SELECT c.country_name FROM countries c
JOIN venues v ON c.country_code = v.country_code
JOIN events e ON v.venue_id = e.venue_id
WHERE e.title = 'LARP Club';
```

### 3.

```
ALTER TABLE venues
ADD COLUMN active BOOLEAN DEFAULT TRUE;
```

## Day 2

### 1.

```
CREATE RULE delete_venue
AS ON DELETE TO venues DO INSTEAD
UPDATE venues SET active = false WHERE venue_id = OLD.venue_id;
```

### 2.

```
SELECT * FROM generate_series(1, 12);
```

### 3.

This exercise is fairly difficult. Here are the steps I followed.

1. We want a final result like this (`wom` is the index of the week in the month):

    ```
     wom | sun | mon | tue | wed | thu | fri | sat
    -----+-----+-----+-----+-----+-----+-----+-----
       0 |     |     |     |     |     |     |
       1 |     |     |     |     |     |     |
       2 |     |     |   1 |   1 |     |     |
       3 |   2 |     |     |     |     |     |
       4 |     |     |     |     |     |     |
    ```


2. We know a `crosstab` can achieve the above if we feed it a table like this:

    ```
     wom | dow | count
    -----+-----+-------
       0 |   3 |
       0 |   4 |
       0 |   5 |
       0 |   6 |
       0 |   0 |
       1 |   1 |
       1 |   2 |
       1 |   3 |
       1 |   4 |
       1 |   5 |
       1 |   6 |
       1 |   0 |
       2 |   1 |
       2 |   2 |     1
       2 |   3 |     1
       2 |   4 |
       2 |   5 |
       2 |   6 |
       2 |   0 |
       3 |   1 |
       3 |   2 |
       3 |   3 |
       3 |   4 |
       3 |   5 |
       3 |   6 |
       3 |   0 |     2
       4 |   1 |
       4 |   2 |
       4 |   3 |
    ```

3. So we need to start by listing all the days in the month (we'll pick February 2012 since it has the most events):

    ```
    SELECT date
    FROM generate_series(
        '2012-02-01'::date,
        '2012-02-29'::date,
        '1 day'::interval
    ) AS date;
    ```

    ```
              date
    ------------------------
     2012-02-01 00:00:00+01
     2012-02-02 00:00:00+01
     2012-02-03 00:00:00+01
     2012-02-04 00:00:00+01
     2012-02-05 00:00:00+01
     2012-02-06 00:00:00+01
     2012-02-07 00:00:00+01
     2012-02-08 00:00:00+01
     2012-02-09 00:00:00+01
     2012-02-10 00:00:00+01
     2012-02-11 00:00:00+01
     2012-02-12 00:00:00+01
     2012-02-13 00:00:00+01
     2012-02-14 00:00:00+01
     2012-02-15 00:00:00+01
     2012-02-16 00:00:00+01
     2012-02-17 00:00:00+01
     2012-02-18 00:00:00+01
     2012-02-19 00:00:00+01
     2012-02-20 00:00:00+01
     2012-02-21 00:00:00+01
     2012-02-22 00:00:00+01
     2012-02-23 00:00:00+01
     2012-02-24 00:00:00+01
     2012-02-25 00:00:00+01
     2012-02-26 00:00:00+01
     2012-02-27 00:00:00+01
     2012-02-28 00:00:00+01
     2012-02-29 00:00:00+01
    ```

2. Then join with the events table:

    ```
    SELECT date, title
    FROM (
        SELECT date
        FROM generate_series(
            '2012-02-01'::date,
            '2012-02-29'::date,
            '1 day'::interval
        ) AS date
    ) AS dates
    LEFT JOIN events
    ON date = date_trunc('day', starts);
    ```

    ```
              date          |      title
    ------------------------+-----------------
     2012-02-01 00:00:00+01 |
     2012-02-02 00:00:00+01 |
     2012-02-03 00:00:00+01 |
     2012-02-04 00:00:00+01 |
     2012-02-05 00:00:00+01 |
     2012-02-06 00:00:00+01 |
     2012-02-07 00:00:00+01 |
     2012-02-08 00:00:00+01 |
     2012-02-09 00:00:00+01 |
     2012-02-10 00:00:00+01 |
     2012-02-11 00:00:00+01 |
     2012-02-12 00:00:00+01 |
     2012-02-13 00:00:00+01 |
     2012-02-14 00:00:00+01 | Valentine's Day
     2012-02-15 00:00:00+01 | LARP Club
     2012-02-16 00:00:00+01 |
     2012-02-17 00:00:00+01 |
     2012-02-18 00:00:00+01 |
     2012-02-19 00:00:00+01 |
     2012-02-20 00:00:00+01 |
     2012-02-21 00:00:00+01 |
     2012-02-22 00:00:00+01 |
     2012-02-23 00:00:00+01 |
     2012-02-24 00:00:00+01 |
     2012-02-25 00:00:00+01 |
     2012-02-26 00:00:00+01 | Wedding
     2012-02-26 00:00:00+01 | Dinner with Mom
     2012-02-27 00:00:00+01 |
     2012-02-28 00:00:00+01 |
     2012-02-29 00:00:00+01 |
    ```

3. Show the count instead of the title:

    ```
    SELECT date, NullIf(count(events), 0) AS count
    FROM (
        SELECT date
        FROM generate_series(
            '2012-02-01'::date,
            '2012-02-29'::date,
            '1 day'::interval
        ) AS date
    ) AS dates
    LEFT JOIN events
    ON date = date_trunc('day', starts)
    GROUP BY date
    ORDER BY date;
    ```

    ```
              date          | count
    ------------------------+-------
     2012-02-01 00:00:00+01 |
     2012-02-02 00:00:00+01 |
     2012-02-03 00:00:00+01 |
     2012-02-04 00:00:00+01 |
     2012-02-05 00:00:00+01 |
     2012-02-06 00:00:00+01 |
     2012-02-07 00:00:00+01 |
     2012-02-08 00:00:00+01 |
     2012-02-09 00:00:00+01 |
     2012-02-10 00:00:00+01 |
     2012-02-11 00:00:00+01 |
     2012-02-12 00:00:00+01 |
     2012-02-13 00:00:00+01 |
     2012-02-14 00:00:00+01 |     1
     2012-02-15 00:00:00+01 |     1
     2012-02-16 00:00:00+01 |
     2012-02-17 00:00:00+01 |
     2012-02-18 00:00:00+01 |
     2012-02-19 00:00:00+01 |
     2012-02-20 00:00:00+01 |
     2012-02-21 00:00:00+01 |
     2012-02-22 00:00:00+01 |
     2012-02-23 00:00:00+01 |
     2012-02-24 00:00:00+01 |
     2012-02-25 00:00:00+01 |
     2012-02-26 00:00:00+01 |     2
     2012-02-27 00:00:00+01 |
     2012-02-28 00:00:00+01 |
     2012-02-29 00:00:00+01 |
    ```

4. Show the week of month and day of week instead of the date:

    ```
    SELECT
        extract(week from date) - extract(week from date_trunc('month', date)) AS wom,
        extract(dow from date) AS dow,
        NullIf(count(events), 0) AS count
    FROM (
        SELECT date
        FROM generate_series(
            '2012-02-01'::date,
            '2012-02-29'::date,
            '1 day'::interval
        ) AS date
    ) AS dates
    LEFT JOIN events
    ON date = date_trunc('day', starts)
    GROUP BY date
    ORDER BY date;
    ```

    ```
     wom | dow | count
    -----+-----+-------
       0 |   3 |
       0 |   4 |
       0 |   5 |
       0 |   6 |
       0 |   0 |
       1 |   1 |
       1 |   2 |
       1 |   3 |
       1 |   4 |
       1 |   5 |
       1 |   6 |
       1 |   0 |
       2 |   1 |
       2 |   2 |     1
       2 |   3 |     1
       2 |   4 |
       2 |   5 |
       2 |   6 |
       2 |   0 |
       3 |   1 |
       3 |   2 |
       3 |   3 |
       3 |   4 |
       3 |   5 |
       3 |   6 |
       3 |   0 |     2
       4 |   1 |
       4 |   2 |
       4 |   3 |
    ```

5. We're now ready for the `crosstab`:

    ```
        SELECT * from crosstab(
           'SELECT
                extract(week from date) - extract(week from date_trunc(''month'', date)) AS wom,
                extract(dow from date) AS dow,
                NullIf(count(events), 0) AS count
            FROM (
                SELECT date
                FROM generate_series(
                    ''2012-02-01''::date,
                    ''2012-02-29''::date,
                    ''1 day''::interval
                ) AS date
            ) AS dates
            LEFT JOIN events
            ON date = date_trunc(''day'', starts)
            GROUP BY date
            ORDER BY date',
           'SELECT * from generate_series(0, 6)'
        ) AS (
            wom int,
            sun int, mon int, tue int, wed int, thu int, fri int, sat int
        )
        ORDER BY wom;
    ```

    ```
     wom | sun | mon | tue | wed | thu | fri | sat
    -----+-----+-----+-----+-----+-----+-----+-----
       0 |     |     |     |     |     |     |
       1 |     |     |     |     |     |     |
       2 |     |     |   1 |   1 |     |     |
       3 |   2 |     |     |     |     |     |
       4 |     |     |     |     |     |     |
    (5 rows)
    ```

    Yay!

## Day 3

### 1.

```
CREATE OR REPLACE FUNCTION suggestion(input text)
RETURNS SETOF text as $$
BEGIN
  RETURN QUERY
  SELECT m.title
  FROM movies m,
       (SELECT genre FROM movies WHERE title ILIKE input) g,
       cube_distance(m.genre, g.genre) dist
  ORDER BY dist
  LIMIT 5;

  IF FOUND THEN RETURN; END IF;

  RETURN QUERY
  SELECT m.title
  FROM movies m
  NATURAL JOIN movies_actors
  NATURAL JOIN actors a
  WHERE a.name ILIKE input
  LIMIT 5;
END;
$$ LANGUAGE plpgsql;
```

### 2.

Ignored the last-name-only requirement. Splitting full names into first and last names is not reliable anyway.

```
CREATE TABLE comments (comment text);

INSERT INTO comments (comment) VALUES
    ('I love Bruce Willis!'),
    ('Anthony Hopkins is the best'),
    ('Meh, don''t like Bruce Willis');

SELECT a.name, count(*)
FROM actors a
JOIN comments c ON c.comment @@ plainto_tsquery(a.name)
GROUP BY a.name
ORDER BY count DESC;
```
