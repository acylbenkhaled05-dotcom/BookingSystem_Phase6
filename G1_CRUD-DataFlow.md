# G1_CRUD_DataFlow.md

## 1️⃣ CREATE – Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (form.js and resources.js)
    participant B as Backend (Express Route)
    participant V as express-validator
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Submit form
    F->>F: Client-side validation
    F->>B: POST /api/resources (JSON)

    B->>V: Validate request
    V-->>B: Validation result

    alt Validation fails
        B-->>F: 400 Bad Request + errors[]
        F-->>U: Show validation message
    else Validation OK
        B->>S: create Resource(data)
        S->>DB: INSERT INTO resources
        DB-->>S: Result / Duplicate error

        alt Duplicate
            S-->>B: Duplicate detected
            B-->>F: 409 Conflict
            F-->>U: Show duplicate message
        else Success
            S-->>B: Created resource
            B-->>F: 201 Created
            F-->>U: Show success message
        end
    end
```

## 2️⃣ READ – Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (resources.js)
    participant B as Backend (Express Route)
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Load /resources page
    F->>B: GET /api/resources

    alt Database error
        B->>S: getAllResources()
        S->>DB: SELECT * FROM resources
        DB-->>S: Error
        S-->>B: Database error
        B-->>F: 500 Internal Server Error
        F-->>U: Show error message
    else Success
        B->>S: getAllResources()
        S->>DB: SELECT * FROM resources
        DB-->>S: Resources list
        S-->>B: Formatted resources
        B-->>F: 200 OK + resources[]
        F-->>U: Display resource list
    end
```

## 3️⃣ UPDATE – Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (form.js and resources.js)
    participant B as Backend (Express Route)
    participant V as express-validator
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Click "Edit" on resource
    F-->>U: Show pre-filled form
    U->>F: Modify fields + click "Update"
    F->>F: Client-side validation

    alt Validation fails
        F-->>U: Show validation message
    else Validation OK
        F->>B: PUT /api/resources/:id (JSON)

        B->>V: Validate request
        V-->>B: Validation result

        alt Validation fails
            B-->>F: 400 Bad Request + errors[]
            F-->>U: Show validation message
        else Validation OK
            B->>S: update Resource(id, data)
            S->>DB: SELECT * FROM resources WHERE id = ?
            DB-->>S: Resource exists?

            alt Resource not found
                S-->>B: Resource not found
                B-->>F: 404 Not Found
                F-->>U: Show "Resource not found" message
            else Resource exists
                S->>DB: UPDATE resources SET ... WHERE id = ?
                DB-->>S: Updated resource
                S-->>B: Success
                B-->>F: 200 OK + updated resource
                F->>B: GET /api/resources (refresh list)
                B-->>F: 200 OK + resources[]
                F-->>U: Show updated list
            end
        end
    end
```

## 4️⃣ DELETE – Resource (Sequence Diagram)

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant F as Frontend (resources.js)
    participant B as Backend (Express Route)
    participant S as Resource Service
    participant DB as PostgreSQL

    U->>F: Click "Delete" on resource
    F-->>U: Show confirmation dialog

    alt User cancels
        F-->>U: Nothing changes
    else User confirms
        F->>B: DELETE /api/resources/:id

        B->>S: delete Resource(id)
        S->>DB: SELECT * FROM resources WHERE id = ?
        DB-->>S: Resource exists?

        alt Resource not found
            S-->>B: Resource not found
            B-->>F: 404 Not Found
            F-->>U: Show "Resource not found" message
        else Resource exists
            S->>DB: DELETE FROM resources WHERE id = ?
            DB-->>S: Resource deleted
            S-->>B: Success
            B-->>F: 204 No Content
            F->>B: GET /api/resources (refresh list)
            B-->>F: 200 OK + resources[]
            F-->>U: Display updated list
        end
    end
```
