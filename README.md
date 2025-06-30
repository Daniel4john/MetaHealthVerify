

# MetaHealthVerify - Healthcare Product Tracking Smart Contract

## Overview

`MetaHealthVerify` is a Clarity smart contract designed for healthcare product lifecycle management and validation tracking. It enables authorized entities to onboard healthcare products, update their production phases, and attach third-party validations securely on the blockchain. This system ensures transparent, auditable tracking of medical product statuses and certifications.

---

## Features

* **Product Lifecycle Management:** Track healthcare products through predefined phases — Produced, Evaluation, Active, and Serviced.
* **Third-Party Validations:** Oversight entities can validate products against multiple standards (health admin, European, quality, security).
* **Access Control:** Only authorized administrators and creators can update product phases.
* **Oversight Entity Management:** Admins can register entities authorized to provide validations.
* **Immutable Timeline:** Maintains a time-stamped history of all product phase changes.
* **Validation Management:** Attach or withdraw validations with clear authorization rules.

---

## Constants

### Product Phases

| Constant           | Value | Description                              |
| ------------------ | ----- | ---------------------------------------- |
| `PHASE_PRODUCED`   | `1`   | Product has been produced.               |
| `PHASE_EVALUATION` | `2`   | Product is under evaluation.             |
| `PHASE_ACTIVE`     | `3`   | Product is active and in use.            |
| `PHASE_SERVICED`   | `4`   | Product is being serviced or maintained. |

### Validation Types

| Constant                       | Value | Description                              |
| ------------------------------ | ----- | ---------------------------------------- |
| `VALIDATION_TYPE_HEALTH_ADMIN` | `1`   | Validated by health administration.      |
| `VALIDATION_TYPE_EUROPEAN`     | `2`   | Validated by European regulatory bodies. |
| `VALIDATION_TYPE_QUALITY`      | `3`   | Quality validation certification.        |
| `VALIDATION_TYPE_SECURITY`     | `4`   | Security validation certification.       |

---

## Error Codes

| Code                                | Meaning                                              |
| ----------------------------------- | ---------------------------------------------------- |
| `ERR_NO_PERMISSION` (1)             | Sender is not authorized to perform the action.      |
| `ERR_PRODUCT_NOT_FOUND` (2)         | Product ID does not exist.                           |
| `ERR_PHASE_UPDATE_FAILED` (3)       | Failed to update the product phase.                  |
| `ERR_INVALID_PHASE` (4)             | Provided phase is not valid.                         |
| `ERR_INVALID_VALIDATION` (5)        | Validation type is invalid or not recognized.        |
| `ERR_VALIDATION_ALREADY_EXISTS` (6) | Validation for this product and type already exists. |

---

## Data Structures

* **`product-data`**: Maps a product ID to its creator, current phase, and a timeline of phases with timestamps.
* **`product-validations`**: Maps a product and validation type to validation info, including the validator, timestamp, and active status.
* **`oversight-entities`**: Tracks authorized entities allowed to validate products for specific validation types.

---

## Roles and Permissions

* **Admin** (contract deployer):

  * Can onboard products at any phase.
  * Can add oversight entities authorized to validate products.
  * Can revoke validations.
* **Product Creator**:

  * Can onboard products but only starting at `PHASE_PRODUCED`.
  * Can update product phase for their own products.
* **Oversight Entities**:

  * Authorized entities that can attach validations to products.
  * Cannot onboard products or modify phases.

---

## Public Functions

### `onboard-product(product-id, initial-phase) -> (response bool uint)`

Registers a new healthcare product with a given ID and starting phase.

* Only admin or sender onboarding at `PHASE_PRODUCED` allowed.
* Product ID must be valid (1 to 1,000,000).
* Phase must be one of the defined phases.

### `modify-product-phase(product-id, new-phase) -> (response bool uint)`

Updates the lifecycle phase of an existing product.

* Only admin or product creator can update.
* Validates phase and product existence.
* Adds new phase event to product timeline.

### `add-oversight-entity(entity, validation-type) -> (response bool uint)`

Adds an authorized oversight entity allowed to attach validations of a specific type.

* Only admin can perform.
* Entities cannot be admin, sender, or zero address.
* Validation type must be valid.

### `attach-validation(product-id, validation-type) -> (response bool uint)`

Attaches a validation to a product by an authorized oversight entity.

* Validates product and validation type.
* Ensures sender is authorized for this validation type.
* Prevents duplicate validations of the same type on the same product.

### `withdraw-validation(product-id, validation-type) -> (response bool uint)`

Deactivates an existing validation on a product.

* Only admin or validator who attached can withdraw.
* Marks validation as inactive but keeps record.

---

## Read-Only Functions

### `retrieve-product-timeline(product-id) -> (response (list 10 {phase, moment}) uint)`

Returns the time-stamped timeline of phases for a product.

### `get-product-phase(product-id) -> (response uint uint)`

Returns the current phase of a product.

### `confirm-validation(product-id, validation-type) -> (response bool uint)`

Checks if a validation is currently active for the product.

### `get-validation-details(product-id, validation-type) -> (response {validator, moment, active} uint)`

Returns details of a specific validation.

### `is-admin(caller) -> (bool)`

Returns true if the caller is the contract administrator.

---

## Internal Logic

* The contract maintains a **moment counter** as a simple timestamp mechanism incrementing on each relevant action.
* Validates product IDs are within range.
* Validates phases and validation types against predefined constants.
* Limits the product timeline to the last 10 phase changes.
* Prevents unauthorized access or tampering.

---

## Usage Example

1. **Admin onboard a product:**

```clarity
(onboard-product 123 u1) ;; Product 123 onboarded at PHASE_PRODUCED by admin
```

2. **Product creator updates product phase:**

```clarity
(modify-product-phase 123 u2) ;; Update phase to PHASE_EVALUATION
```

3. **Admin adds oversight entity:**

```clarity
(add-oversight-entity 'SP3...xyz u1) ;; Authorize entity for health admin validations
```

4. **Oversight entity attaches validation:**

```clarity
(attach-validation 123 u1) ;; Attach health admin validation to product 123
```

5. **Query product timeline:**

```clarity
(retrieve-product-timeline 123) ;; Returns timeline of phases
```

---
