# Notes

Use this file as a scratchpad to document examples of API requests/responses and Table Structures.

### DB Strategies

---

### 1. **Entity Attribute Value (EAV)**

Single extra table which stores the entity (record in another table), attribute title, and value. Can include multiple value types for data validation.

![EAV Diagram](images\eav2.png)

| Pros                                                                | Cons                                                                                                                                         |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| <ul><li>Less space</li><li>Flexible</li><li>Simple design</li></ul> | <ul><li>Poor performance with more data</li><li>Can't effectively index columns</li><li>Complex queries</li><li>No data validation</li></ul> |

//Relational DBs don't support inheritance -> Three methods to address this:

### 2. **Single Table Inheritance**

"Represents an inheritance hierarchy of classes as a single table that has columns for all the fields of the various classes."

Add all columns you need to main table. Pre-determine what type of data you might need.

| Pros                                                                                                | Cons                                                                       |
| --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| <ul><li>Easy to query and index</li><li>Good performance</li><li>Data validation possible</li></ul> | <ul><li>Hard to work with all columns</li><li>Not truly flexible</li></ul> |

### 3. **Class Table Inheritance**

Pre-defined Attributes

![Defined Diagram](images\fixed_attr.png)

"Represents an inheritance hierarchy of classes with one table for each class."

General record type and seperate tables for each sub type with specific attributes and relate back to the main table.

| Pros                                                                                                                                              | Cons                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| <ul><li>All columns in DB already</li><li>Smaller tables compared to Single table</li><li>Data Validation</li><li>Increased Performance</li></ul> | <ul><li>Not flexible, must use existing columns</li><li>Multiple tables become confusing</li></ul> |

### 4. **Concrete Table Inheritance**

"Represents an inheritance hierarchy of classes with one table per concrete class in the hierarchy."

No main/central table to reference back to. Each table type contains all the attributes it needs.

| Pros                                                                                                                    | Cons                                                                                |
| ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| <ul><li>Good data validation</li><li>Good performance with indexing</li><li>Fewer joins with no central table</li></ul> | <ul><li>Not flexible</li><li>Extra logic to handle different record types</li></ul> |

### 5. **Normalized Tables**

Seperate look up tables for each type of data. Must define up front.

| Pros                                                                                                                                               | Cons                           |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| <ul><li>Uses DB features in best way</li><li>Allows for relationships</li><li>Reduce redundancy</li><li>Data validations and constraints</li></ul> | <ul><li>Not flexible</li></ul> |

### 6. **JSON/JSONB Field (Or XML)**

Define JSON field in custom attributes field in employees table.

![JSON Diagram](images\json.png)

| Pros                                                                                                                                                                                                  | Cons                                                                                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <ul><li>Functions to process JSON have improved</li><li>Simple Design</li><li>Truely Flexible</li><li>Indexing on JSONB</li><li>Support for indexing and querying - if a little more awkard</li></ul> | <ul><li>Hard to enforce data validations</li><li>Potential to get messy</li><li>Misses the point of a relational database - to enforce structure via schemas</li></ul> |

### 6. **Dynamic Schema**

Utilize ALTER TABLE statement to add the column, and data type, to table.

![Dynamic Diagram](images\dynamic.png)

| Pros                                                                               | Cons                                                                                                                                                                                                 |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <ul><li>Flexible</li><li>Keep use of DB features</li><li>Data validation</li></ul> | <ul><li>Risky to modify on live system</li><li>Extra logic required to do correctly</li><li>All instances get all fields, so is less handy if you get a lot of variance between instances.</li></ul> |

### 7. **NoSQL DB**

| Pros                                                                                                                                             | Cons                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------- |
| <ul><li>Flexible</li><li>Ability to scale across multiple servers</li><li>Ability to gradually migrate data</li><li>Fast with no joins</li></ul> | <ul><li>No data validation</li><li>Redundancy</li></ul> |

"When using a schemaless approach, consider using a clear data access layer and/or a predicate schema to reduce the problems of a hidden, implicit schema." Martin Fowler

### Database Tables

```

```

### CRUD:

```SQL
POST /businesses/:business_id/custom_attributes
PUT/POST /businesses/:business_id/custom_attributes
GET /businesses/:business_id/custom_attributes
DELETE /businesses/:business_id/custom_attributes
```

### Employees to specify their values:

POST /employees/:employee_id

```json
{
  "sales_ranking": 2,
  "email": "employee@business.com"
}
```

```SQL

 UPDATE employees
 SET custom_attributes = jsonb_set(custom_attributes, '{sales_ranking}', '3', true)
 WHERE custom_attributes ->> 'internal_id' = 'ab678936jh';

```

### Create new custom field:

```SQL

 UPDATE employees
 SET custom_attributes = jsonb_set_lax(custom_attributes, '{newAttribute}', NULL, true, 'use_json_null')
 WHERE business_id = 8;
```

### Trip custom fields:

Same options as for employee custom fields

### Multiple choice fields:

```json
{
  "cost_centers": {
    "process": 0,
    "personal": 1,
    "impersonal": 0,
    "production": 1
  },

  "cost_centers": [0, 1, 0, 1]
}
```

### Performance:

Default GIN supports @>, ?, ?& and ?| operators.

```SQL

CREATE INDEX idxgin ON employees USING gin (custom_attributes);

SELECT custom_attributes->'sales_ranking', custom_attributes->'demerits' FROM employees WHERE custom_attributes @> '{"internal_employee_id": "773663hgksh"}';

```

Non-default jsonb_path_ops supports indexing the @> operator only.

Pros:

- Each jsonb_path_ops index item is a hash of the value and the key(s) leading to it
- Specific and fast.
- Notable performance advantages

Cons:

- No way just to search by key only.

- Will not index empty value key/value pairs.

### Parameter Limitations

```SQL
GET
/businesses/:business_id/employees?date=
```

### Data Validation

- Allow SQL to validate data
- Business logic to validate data. Interface, Data Access Layer...

### Security

- Secure API with OAuth
- API key to request a token
- Token redirects user to authentication partner
- Third party confirms authorization
- Request access aoken
- Return access token with timeout

### PostgreSQL

- Consider breaking out common custom attributes.

#### JSONB

- Does not preserve white space or numbers entered with E notation

- Will preserve trailing zeros

- Containment and Existence operators

- functions and operators for processing and creating JSON data

- SQL/JSON path language

- Recommended that JSON Documents have a somewhat fixed structure

- Limit JSON documents to manageble size to decrease lock contention among updating transactions
