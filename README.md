# [Application Programming] Group 9 Project

## Team

| First name | Last name | Student ID | University email address |
| --- | --- | --- | --- |
| Adam | Wasiak | 343076 | adam.wasiak2.stud@pw.edu.pl |
| Andrzej | Buczma | 343004 | ??? |
| Krystian | Kopeć | 337570 | ??? |
| Oskar | Maj | 343033 | ??? |

## Project Topic

### Problem

Beginners, hobbyists, and engineers building IoT prototypes lose hours selecting compatible components. Browsing thousands of datasheets leads to decision-making errors: incorrect voltages (a burned circuit), missing resistors, or incompatible connectors. Traditional stores, on the other hand, lack project-specific guidance.

### Solution

A web application that, based on a description in natural language (either a ready-made plan or one developed during a conversation, e.g. "a battery-powered weather station with phone notifications"), designs the hardware architecture, verifies physical compatibility, and creates a ready-to-use shopping cart (links for purchasing individual components).

### Functionality

> Fragments marked this way represent optional functionality or functionality requiring the selection of one or more options during further planning.

- A chat connected to an LLM that, after the target parts supplier is selected:
    - Learns the project's goal from the user (receives a ready-made description or reaches the goal through questions)
    - Plans which component categories and quantities will be required
    - Supports two ways of querying the parts database (with availability and supplier filtering):
        > - vector search using embeddings in the database for each requested component, with category filtering
        > - returning $x$ results from each required category, allowing the LLM to choose from each large category list independently
    - Deterministic rules *manually* added to the database verify engineering requirements (3.3V vs. 5V voltage levels, current balance, I2C buses). If a sensor requires 5V and the microcontroller uses 3.3V, the system automatically adds a voltage converter to the cart.
    - A shopping cart (a list of links) is prepared from the collected parts and saved in the database
    - The conversation history is saved in the database
    - > Optionally, the user's project can include generated and saved sample wiring diagrams and/or sample microcontroller code
- The user registers for and logs in to a private account, gaining access to the chat and project history. The user can change their email address and password.
- The application stores the history of projects for which the user searched for parts (the conversation, found purchase links, and other generated elements, if any)
- The application may perform an internal **migration of data from an API to the database**, which involves querying several categories for a specified number of products (migration from a CSV file may also be supported). Several approaches are possible (some dependent on the selected parts-search method):
    > - each product returned by the API is sent to the LLM, which generates its product description; an embedding is then created from that description. The LLM can also translate API results into an established internal data format (e.g. after receiving the *provider_shop_link_purchase* field in JSON, it places it in the internally defined *purchase_link* field in the database; this makes the system independent of a single API because it supports any data format returned by an API).
    > - the project depends on a single API with a fixed response format, and data is deterministically inserted into the appropriate database fields without LLM processing
    > - the project depends on a single API with a fixed response format, and data is deterministically inserted into the appropriate database fields while an LLM description and embedding are also generated
    Each product receives one record in the database.
- The application may perform an internal **product availability refresh**, which involves sending one of the following selectable types of queries and updating the product's availability status:
    > - for every available product
    > - only for products with the lowest available quantity (below a specified threshold)
- An administrator account allowing the user to run data migrations from the API to the database, refresh product availability, and update engineering rules
- > The administrator account allows access to general statistics:
    > - History of data migrations from the API to the database, including individual dates, statuses, and numbers of indexed products
    > - History of product availability refreshes, including individual dates, statuses, and numbers of updated products
    > - The number of users and the number/average number of conversations they have created

### Architecture

Architectural style: SPA + REST API + Relational-vector database

> Does migrating data from the API to the database or refreshing product availability require a task queue and workers to process the tasks?

#### Frontend

A web application divided into:
- login and registration screens
- regular user access:
    - chat
    - conversation history
    - account management panel
- administrator panel:
    - running migrations
    - statistics
    - updating engineering rules

#### Backend

REST API divided into modules:
- user accounts (login, registration, password/email changes)
- chat (conversations with the LLM)
- conversation history
- data migration from the API to the database
- product availability
- > statistics

### Technologies

- SPA: **React**
- > Component library for the SPA: ???
- REST API: **Java** + framework: **Spring Boot**
- Relational-vector database: **PostgreSQL + pgvector**
- Data API: **nexar.com**
- LLM provider: **OpenAI**
- ORM: **Hibernate**
- > Login and registration: Keycloak/Clerk???
- Containerization: **Docker, Docker Compose**
- > API build automation: Maven/Gradle???
- Requests to the LLM provider: **Spring AI**
