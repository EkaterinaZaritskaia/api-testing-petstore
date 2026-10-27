Base URL: `http://localhost:8080/api/v3`
Expected results are taken from the OpenAPI schema (`openapi.json`), not from the API's actual behaviour.

## Pet CRUD

| ID | Area | Test case | Technique | Data | Expected | Actual                   | Result                                                                                  |
|---|---|---|---|---|---|--------------------------|-----------------------------------------------------------------------------------------|
| P-01 | Pet CRUD | Create a valid pet (`POST /pet`) | positive | id 5001, name Rex, photoUrls [], status available | 200, body echoes the pet | 200, body echoes the pet | pass                                                                                    |
| P-02 | Pet CRUD | Get the created pet (`GET /pet/{petId}`) | lifecycle | id 5001 | 200, body has the same id and name | 200                      | pass                                                                                    |
| P-03 | Pet CRUD | Update the pet name (`PUT /pet`) | lifecycle | id 5001, name Max | 200, body has name Max | 200                      | pass                                                                                    |
| P-04 | Pet CRUD | Delete the pet (`DELETE /pet/{petId}`) | lifecycle | id 5001 | 200 (success not documented, schema lists only 400) | 200                      | question                                                                                |
| P-05 | Pet CRUD | Get the deleted pet (`GET /pet/{petId}`) | lifecycle | id 5001 | 404 | 404                      | pass                                                                                    |
| P-06 | Pet CRUD | Update a non-existent pet (`PUT /pet`) | negative | id 5001 (deleted), name Max, photoUrls [] | 404 | BUG                      | 200, a new pet 5001 is created                                                          |
| P-07 | Pet CRUD | Delete the already deleted pet (`DELETE /pet/{petId}`) | lifecycle | id 5001 | 404 (schema lists only 400) | 200 | pass (documents only 400. Success (200) and not found (404) are missing in the schema.) |
| P-08 | Pet CRUD | Update name with query parameters (`POST /pet/{petId}`) | positive | id 1, name=Max | 200 (success not documented, schema lists only 405) | 200 | question                                                                                |

## Pet search

| ID | Area | Test case | Technique | Data | Expected | Actual                                                 | Result |
|---|---|---|---|---|---|--------------------------------------------------------|--------|
| P-09 | Pet search | Find pets with status available (`GET /pet/findByStatus`) | equivalence classes | status=available | 200, every pet in the list has status available | 200, every pet in the list has status available        | pass   |
| P-10 | Pet search | Find pets with status pending (`GET /pet/findByStatus`) | equivalence classes | status=pending | 200, every pet in the list has status pending | 200, every pet in the list has status pending          | pass   |
| P-11 | Pet search | Find pets with status sold (`GET /pet/findByStatus`) | equivalence classes | status=sold | 200, every pet in the list has status sold | 200, every pet in the list has status sold             | pass   |
| P-12 | Pet search | Find pets by a status not in the enum (`GET /pet/findByStatus`) | equivalence classes, negative | status=lost | 400 Invalid status value | 200, empty list []                                     | BUG    |
| P-13 | Pet search | Find pets without the status parameter (`GET /pet/findByStatus`) | equivalence classes, negative | no status parameter | 200, returns available pets (status is optional, default is available) | 200, empty list [] (default "available" is not applied | BUG    |
| P-14 | Pet search | Find pets by tag (`GET /pet/findByTags`) | positive | tags=tag1 | 200, every pet in the list has tag1 in tags | 200                                                    | pass   |

## Pet validation

| ID | Area | Test case | Technique | Data | Expected | Actual | Result |
|---|---|---|---|---|---|---|---|
| P-15 | Pet validation | Create a pet without the name field (`POST /pet`) | required field | {"id": 5002, "photoUrls": []} | 400 or 405 (error; schema documents 405 Invalid input for POST /pet) | | |
| P-16 | Pet validation | Create a pet without the photoUrls field (`POST /pet`) | required field | {"id": 5003, "name": "Bruno"} | 400 or 405 (error; schema documents 405 Invalid input for POST /pet) | | |
| P-17 | Pet validation | Create a pet with a number as name (`POST /pet`) | wrong type | {"id": 5004, "name": 123, "photoUrls": []} | 400 or 405 (error; schema documents 405 Invalid input for POST /pet) | | |
| P-18 | Pet validation | Create a pet with a number in photoUrls (`POST /pet`) | wrong type | {"id": 5005, "name": "Bruno", "photoUrls": [123]} | 400 or 405 (error; schema documents 405 Invalid input for POST /pet) | | |
| P-19 | Pet validation | Get a pet with a letter in the id (`GET /pet/{petId}`) | wrong type | id 1234g567 | 400 Invalid ID supplied | | |
| P-20 | Pet validation | Get a pet with a decimal id (`GET /pet/{petId}`) | wrong type | id 1234.567 | 400 Invalid ID supplied | | |
| P-21 | Pet validation | Create a pet with status not in the enum (`POST /pet`) | equivalence classes, negative | {"id": 5006, "name": "Bruno", "photoUrls": [], "status": "lost"} | 400 or 405 (error; schema documents 405 Invalid input for POST /pet) | | |
| P-22 | Pet validation | Create a pet without id (`POST /pet`) | optional field | {"name": "Bruno", "photoUrls": []} | 200, pet is created with an id assigned by the server | | |
| P-23 | Pet validation | Create a pet with a body that is not JSON (`POST /pet`) | wrong structure | abc | 400 or 405 (error; schema documents 405 Invalid input for POST /pet) | | |


## Store (orders)

| ID | Area | Test case | Technique | Data | Expected | Actual | Result |
|---|---|---|---|---|---|---|---|
| S-01 | Store | Create a valid order (`POST /store/order`) | positive | {"id": 5001, "petId": 1, "quantity": 7} | 200, body echoes the order | | |
| S-02 | Store | Get the created order (`GET /store/order/{orderId}`) | lifecycle | orderId 5001 | 200, body has the same id and petId | | |
| S-03 | Store | Delete the order (`DELETE /store/order/{orderId}`) | lifecycle | orderId 5001 | 200 (success not documented, schema lists only 400, 404) | | |
| S-04 | Store | Get the deleted order (`GET /store/order/{orderId}`) | lifecycle | orderId 5001 | 404 Order not found | | |
| S-05 | Store | Delete the already deleted order (`DELETE /store/order/{orderId}`) | lifecycle | orderId 5001 | 404 Order not found | | |
| S-06 | Store | Create an order with a negative quantity (`POST /store/order`) | boundary values | {"id": 5002, "petId": 1, "quantity": -7} | 200 per schema (no minimum for quantity); business rule unclear | | |
| S-07 | Store | Create an order for a non-existent pet (`POST /store/order`) | negative | {"id": 5003, "petId": 198772, "quantity": 7} | Not defined in schema; by business logic an error is expected | | |
| S-08 | Store | Get an order that never existed (`GET /store/order/{orderId}`) | negative | orderId 999999 | 404 Order not found | | |
| S-09 | Store | Get the store inventory (`GET /store/inventory`) | positive | no parameters | 200, body is a map of status → count | | |

## User

| ID | Area | Test case | Technique | Data | Expected | Actual | Result |
|---|---|---|---|---|---|---|---|
| U-01 | User | Create a valid user (`POST /user`) | positive | {"id": 5001, "username": "katya_test", "firstName": "Kate", "password": "123"} | 200, body echoes the user | | |
| U-02 | User | Get the created user (`GET /user/{username}`) | lifecycle | username katya_test | 200, body has the same username | | |
| U-03 | User | Update the user first name (`PUT /user/{username}`) | lifecycle | username katya_test, body {"id": 5001, "username": "katya_test", "firstName": "Katya", "password": "123"} | 200, firstName is Katya | | |
| U-04 | User | Log in with a valid username and password (`GET /user/login`) | positive | username=katya_test&password=123 | 200 | | |
| U-05 | User | Log in with a wrong password (`GET /user/login`) | negative | username=katya_test&password=qwerty | 400 Invalid username/password supplied | | |
| U-06 | User | Log in without parameters (`GET /user/login`) | negative | no parameters | 400 Invalid username/password supplied | | |
| U-07 | User | Log in with empty parameters (`GET /user/login`) | negative | username=&password= | 400 Invalid username/password supplied | | |
| U-08 | User | Delete the user (`DELETE /user/{username}`) | lifecycle | username katya_test | 200 (success not documented in schema) | | |
| U-09 | User | Get the deleted user (`GET /user/{username}`) | lifecycle | username katya_test | 404 User not found | | |
| U-10 | User | Delete the already deleted user (`DELETE /user/{username}`) | lifecycle | username katya_test | 404 User not found | | |

## Out of scope

## Questions for the analyst
- POST /pet documents 405 Invalid input for invalid data. Should it be 400? 
- Can an order have quantity 0 or less? The schema has no minimum.
- Must `petId` in an order refer to an existing pet?