# Recipe Box API Security Audit

## 1. Safe Storage

**Protection tested:**
Password storage in the `users` table.

**Evidence:**
I ran:

```sql
SELECT id, username, password_hash
FROM users
LIMIT 5;
```

Some accounts had Werkzeug PBKDF2 password hashes, such as:

```text
pbkdf2:sha256:1000000$...
```

However, some test accounts still contained values beginning with:

```text
PLAINTEXT:...
```

**Conclusion:** **Needs cleanup.** The API supports secure password hashing, but the remaining plaintext test passwords should be migrated to hashed passwords before production use.

## 2. Authentication

**Route tested:**
`POST /recipes`

**Anonymous request result:**
Returned **401 Unauthorized** with:

```text
"error": "Authorization header missing"
```

**Invalid token result:**
Returned **401 Unauthorized** with:

```text
"error": "Invalid token"
```

**Valid token result:**
Returned **201 Created** and successfully created a recipe with `owner_id: 5`.

**Conclusion:** **Pass.** The protected route requires a valid authentication token and rejects missing or invalid tokens.

## 3. Authorization — Ownership & Roles

### Modify — PATCH /recipes/:id

**Non-owner result:**
`testuser` (User ID 3) attempted to modify a recipe owned by User ID 5. The API returned **403 Forbidden**:

```text
"forbidden: you do not have permission to modify this recipe"
```

**Owner result:**
`jessica` (User ID 5) modified the recipe successfully. The API returned **200 OK**.

### Delete — DELETE /recipes/:id

**Non-owner result:**
`testuser` attempted to delete the recipe and received **403 Forbidden**:

```text
"error": "forbidden: you do not have permission to delete this recipe"
```

**Owner result:**
`jessica` deleted the recipe successfully and received **204 No Content**.

**Conclusion:** **Pass.** The API enforces recipe ownership for both modification and deletion. Authenticated users cannot modify or delete another user's recipes, while the owner can perform those actions.

## Overall Audit Conclusion

The Recipe Box API correctly enforces authentication and recipe ownership for the tested routes. Missing or invalid authentication results in 401, while authenticated users without permission receive 403. Owners can successfully modify and delete their own recipes. Password storage still needs cleanup because some existing test accounts contain plaintext passwords.
