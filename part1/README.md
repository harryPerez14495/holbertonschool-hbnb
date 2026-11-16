# HBnB Evolution - Part 1: Technical Documentation

**Harry Perez**  
**Student Holberton**  
**Cohort 30**

## Project Overview

HBnB Evolution is a simplified property-listing application.
Users can register and manage profiles, owners can create places,
users can submit reviews, and amenities describe property features.

Part 1 provides the technical documentation that guides the
implementation of the application. It includes six UML diagrams
and their explanatory notes.

## Architecture

| Layer | Main Components | Responsibility |
| --- | --- | --- |
| Presentation | ServiceAPI | Receive HTTP requests and return responses. |
| Business Logic | HBnBFacade, User, Place, Review, Amenity | Coordinate application operations and enforce business rules. |
| Persistence | Repository, Database | Store and retrieve application data. |

HBnBFacade provides the entry point used by the API to access
business operations. It coordinates domain models and delegates
storage operations to repositories.

## Complete Technical Document

- [PDF edition](HBnB_Technical_Documentation_Harry_Perez.pdf)
- [Markdown edition](technical_documentation.md)

The complete document includes an introduction, architecture,
class design, API interactions, and explanatory notes.

## Source Design Files

- [High-level package diagram](high_level_package_diagram.md)
- [Detailed class diagram](detailed_class_diagram.md)
- [API sequence diagrams](sequence_diagrams.md)

## API Interaction Diagrams

The sequence diagrams cover four use cases:

1. User registration.
2. Place creation.
3. Review submission.
4. Fetching a list of places.

Each diagram shows communication between the application layers,
including success and error paths.

## Design Conventions

- Domain entities use UUID4 identifiers.
- Every entity records creation and update timestamps.
- Passwords are stored as hashes and excluded from public responses.
- Mermaid source diagrams are included in the Markdown files.
- Proposed interfaces and deletion policies are identified for review.
- Database technology will be selected in a later project part.

## Student Information

| Field | Information |
| --- | --- |
| Name | Harry Perez |
| School | Holberton School |
| Role | Student |
| Cohort | 30 |
| GitHub | [harryPerez14495](https://github.com/harryPerez14495) |
