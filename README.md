# SQL Convert (fullstack generator)
This is rewritten version to Python, original version located [here](https://github.com/KrzysztofProgrammer/sql_to_fnc)
and was written in TypeScript

Project using DDL CREATE TABLE definition will generate:
* PostgreSQL functions ``_delete``, ``_get``, ``_search``, ``_save`` . 
* NestJS REST API with dto definition, service, controller and module template and basic E2E tests. 
* WWW Angular module with: dto-s, service, routing, edit page and list page with pagination and basic filtering that is remembered


Comments on table column is used for API description and labels for edits. 
Of course, this will not meet in 100% yours needs, but will speed up your development time.

## Usage
! Important ! Python 3.10 or higher needed in your machine, you can download it [here](https://www.python.org/downloads/)

* Before usage please change inside constants.py database username.

*Usage:*
```
py gen.py table.generate.sql
```
or
```
python gen.py table.generate.sql
```

where **table.generate.sql** is file with DDL table definition.

## Generation SQL

Example table.example.sql definition:
```
CREATE TABLE auth.user
(
  id                 integer    NOT NULL DEFAULT NEXTVAL('user_serial'::regclass),
  login              varchar    NOT NULL,
  active             boolean    NOT NULL DEFAULT true,
  password           varchar    NOT NULL,
  notes              varchar
);
COMMENT ON COLUMN auth.user.login IS 'Login, minimum 3 letters';
COMMENT ON COLUMN auth.user.active IS 'Is account active?';
COMMENT ON COLUMN auth.user.password IS 'Please use minimum 8 letters, upper and lower case, number';
COMMENT ON COLUMN auth.user.notes IS 'Admin notes about user';
```
Main assumptions:
* file should have only table definition
* table name should have schema information. For public will be generated automatically
* table name should contain only small letters
* table name should not be in plural (wrong table name: users)
* primary key is first element, number type with NEXTVAL() definition
* second element (in this case - login) will be searchable at list method (inside filter)


## Generation SQL functions

It will create files in dist/sql directory with functions :
* auth.user_delete(id integer)
* auth.user_get(id integer)
* auth.user_search(filter varchar) - filter is prepared for Angular Materials Server Side pagination
* auth.user_save(user varchar) - parameter is JSONB object cast on varchar. If id more than zero - will update data, else - will create new record.


## Generation NestJS REST API

Using Node.js Style Guide from https://github.com/airbnb/javascript and https://github.com/felixge/node-style-guide 

It will generate files:
* dist/api/shared/dto/FilterItem.dto.ts - global definition for filter item, used in ListFilterRequest 
* dist/api/shared/dto/ListFilterRequest.dto.ts - global definition request for list api
* dist/api/customer/dto/User.dto.ts  - with type definition for user
* dist/api/customer/dto/UserFilter.dto.ts - list filter. Example: ```{"filter": [{"field": "login", "value": "Joh%"}], "sort": ["login"], "page_size": 25, "page_index": 1, "sort_direction": "asc"}```
* dist/api/customer/dto/UserListResponse.dto.ts  - response from list function with table count information and data array - filtered user table
* dist/api/customer/user.controller.ts - with API endpoints: POST /list, GET :id, POST /, DELETE :id
* dist/api/customer/user.service.ts - with methods covering controller above definitions: list, save, get, delete. All methods call SQL functions.

## Generation E2E supertest files

It will generate:
* dist/tests/data/user.data.ts - contain valid user data, invalid user data and filter data. This should be filled by needs.
* dist/tests/user.e2e-spec.ts - contain supertest template with test cases:
   * POST /user/list - with filter data, get first element from list and use it
   * GET /user/{number} - use above data for fetch item
   * POST /user  - save new item with valid data
   * POST /user  - update item with valid data
   * DELETE /user/0 - check wrong data
   * DELETE /user/{number} - delete added above data 

## Generation WWW CRUD Angular module

Using Style Guide by https://angular.io/guide/styleguide

It will generate:
* dist/www/user.datasource.ts - needed for pagination by list.component.ts
* dist/www/user.module.ts - Angular module for dynamic loading in main route module
* dist/www/user.service.ts - mapping generated by swagger API module to URL provided in .env file (apiUrl)
* dist/www/user-routing.module.ts - routing module for list, edit items
* dist/www/list - directory with list with filter item component:
  * list.component.html
  * list.component.scss
  * list.component.ts
* dist/www/edit - directory with edit item component:
  * edit.component.html 
  * edit.component.scss
  * edit.component.ts
