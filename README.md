# 


# Router
```bash
go get github.com/julienschmidt/httprouter
go install github.com/cortesi/modd/cmd/modd@latest
```


## JSON

### Sending Response (json encoding)
- *json.Marshal()
- *Envelop
- *Helper function to write JSON to response
- *Customize tag with golang struct tags
- Advanced customize: MarshalJSON() implementation
- *Sending Error Message

```bash
curl localhost:4000/v1/movies/1
```
### Parsing Request (json decoding)
- *json decode
- *Bad requests:
  - json.SyntaxError, io.ErrUnexpectedEOF
  - json.UnmarshalTypeError
  - json.InvalidUnmarshalError
  - io.EOF
```bash
BODY='{"title":"Moana","year":2016,"runtime":107, "genres":["animation","adventure"]}'
curl -i -d "$BODY" localhost:4000/v1/movies

curl -d '{"title": "Ok",}' localhost:4000/v1/movies

```
- *Restricting input
```bash
curl -i -d '{"title": "Moana"}{"title": "Top Gun"}' localhost:4000/v1/movies
curl -i -d '{"title": "Moana"} :~()' localhost:4000/v1/movies 
curl -d '{"title": "Moana", "rating":"PG"}' localhost:4000/v1/movies
```
- Custom JSON Decoding
```bash
curl -d '{"title": "Moana", "runtime": "107 mins"}' localhost:4000/v1/movies
```

- *Validate JSON Input
```bash
BODY='{"title":"","year":1000,"runtime":"-123 mins","genres":["sci-fi","sci-fi"]}'
curl -i -d "$BODY" localhost:4000/v1/movies

# {"error":{"genres":"must not contain duplicate values","runtime":"must be a positive integer","title":"must be provided","year":"must be greater than 1888"}}
```

## Postgres

```psql
CREATE ROLE greenlight WITH LOGIN PASSWORD 'greenlight'
CREATE EXTENSION IF NOT EXISTS citext;
```

```bash
go get github.com/lib/pq
```
dsn="postgres://postgres:postgres@localhost:5432/greenlight"


### SQL Migrations
```bash
brew install golang-migrate

migrate create -seq -ext=.sql -dir=./migrations create_movies_table
migrate create -seq -ext=.sql -dir=./migrations add_movies_check_constraints
migrate -path=./migrations -database="postgres://postgres:postgres@localhost:5432/greenlight?sslmode=disable" up
```
```postgresql
-- create_moviese_table_up.sql
CREATE TABLE IF NOT EXISTS movies (
                                      id bigserial PRIMARY KEY,
                                      created_at timestamp(0) with time zone NOT NULL DEFAULT NOW(),
                                      title text NOT NULL ,
                                      year integer NOT NULL,
                                      runtime integer NOT NULL,
                                      genres text[] NOT NULL,
                                      version integer NOT NULL DEFAULT 1

);

-- create_movies_table_down.sql
DROP TABLE IF EXISTS movies;

-- add_movies_check_constraints.up.sql
ALTER TABLE movies ADD CONSTRAINT movies_runtime_check CHECK (runtime >= 0);

ALTER TABLE movies ADD CONSTRAINT movies_year_check CHECK (year BETWEEN 1888 AND date_part('year', now()));

ALTER TABLE movies ADD CONSTRAINT genres_length_check CHECK (array_length(genres,1) BETWEEN 1 AND 5);
```

### Set up MovieModel and Models (container for MovieModel)
### CRUD Operations
- DB.Query() is used for SELECT queries which return multiple rows.
- DB.QueryRow() is used for SELECT queries which return a single row.
- DB.Exec() is used for statements which don’t return rows (like INSERT and DELETE).

```bash
BODY='{"title":"Moana","year":2016,"runtime":"107 mins", "genres":["animation","adventure"]}'
curl -i -d "$BODY" localhost:4000/v1/movies

BODY='{"title":"Black Panther","year":2018,"runtime":"134 mins","genres":["action","adventure"]}'
curl -i -d "$BODY" localhost:4000/v1/movies

BODY='{"title":"Deadpool","year":2016, "runtime":"108 mins","genres":["action","comedy"]}'
curl -i -d "$BODY" localhost:4000/v1/movies

BODY='{"title":"The Breakfast Club","year":1986, "runtime":"96 mins","genres":["drama"]}'
curl -i -d "$BODY" localhost:4000/v1/movies

```
- Create a new movie
- Fetch a movie
- Update a movie (fully)
- Delete a movie
- Update a movie (partial)
- List all movies:
  - Filtering
  - Sorting
  - Pagination

### Filtering, Sorting and Pagination:
- Parsing Query String Params

- Filter
```bash
curl localhost:4000/v1/movies?title=godfather&genres=crime,drama&page=1&page_size=5&sort=-year

curl http://localhost:4000/v1/movies

# Dynamic filter
curl http://localhost:4000/v1/movies?title=black+panther
curl "localhost:4000/v1/movies?genres=adventure"
curl "localhost:4000/v1/movies?title=moana&genres=animation,adventure"

# Full-text search

curl "localhost:4000/v1/movies?title=the+club"
curl "localhost:4000/v1/movies?title=panther"
```
- Index:
```bash
migrate create -seq -ext=.sql -dir=./migrations add_movies_indexes
migrate -path=./migrations -database="postgres://postgres:postgres@localhost:5432/greenlight?sslmode=disable" up

```
- Sort
```bash
curl "localhost:4000/v1/movies?sort=title"
curl "localhost:4000/v1/movies?sort=-year"
curl "localhost:4000/v1/movies?sort=-title"
```


### JSON Logging

### Rate Limit
```bash
go get golang.org/x/time/rate@latest
```

### Graceful Shutdown


### Authentication
#### Basic authentication
#### Token authentication:
- The client sends a request to your API containing their credentials (username, email, password)
- The server verifies username, passwsord and return the (bearer) token. Token expires after a set period of time.
- Client saves the token locally and use it when requesting API data again.
- The server receives the request, check if the token hasn't expired

* Stateful token: server store the token (or its hash) in a database, alongside the userID and expiry time
* Stateless token (JWT): stateless tokens encode the user ID and expiry time in the token itself.

### UserModel 
```bash
migrate create -seq -ext=.sql -dir=./migrations create_users_table
```

```postgresql

```

### Test permission
```bash
BODY='{"email": "alice@example.com", "password": "pa55word"}'
curl -d "$BODY" localhost:4000/v1/tokens/authentication


curl -H "Authorization: Bearer NNEI2GHRXLO3XHGVCQE3XWKBPY" localhost:4000/v1/movies/1
curl -X DELETE -H "Authorization: Bearer NNEI2GHRXLO3XHGVCQE3XWKBPY" localhost:4000/v1/movies/2

BODY='{"email": "faith@example.com", "password": "pa55word"}'
curl -d "$BODY" localhost:4000/v1/tokens/authentication

curl -H "Authorization: Bearer VYAFPVATASFEZRUXK4XLOIMC2Y" localhost:4000/v1/movies/1
curl -X DELETE -H "Authorization: Bearer VYAFPVATASFEZRUXK4XLOIMC2Y" localhost:4000/v1/movies/1

```